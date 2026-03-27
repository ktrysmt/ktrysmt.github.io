---
layout: post
title: "君はcaffeinateの本当の使い方を知っているか"
date: 2026-03-28 01:00:00 +0900
categories: [Developer Tools]
published: true
description: "macOSのスリープを一時的に抑制するcaffeinateのマニアックな使い方とか。"
tags:
  - macos
  - cli-tools
  - shell
---

いろいろ工夫のしがいがあって結構おもろい。

## オプションまとめ

| フラグ | 対象 | 説明 |
|--------|------|------|
| `-d` | ディスプレイ | ディスプレイスリープを防止 |
| `-i` | システム | システムのアイドルスリープを防止 |
| `-m` | ディスク | ディスクのアイドルスリープを防止 |
| `-s` | システム | AC電源接続時のシステムスリープを防止 |
| `-u` | ディスプレイ | ユーザーがアクティブであると宣言（ディスプレイを起こす） |
| `-t <秒>` | - | 指定秒数後にアサーションを自動解除 |
| `-w <PID>` | - | 指定PIDのプロセスが終了するまでアサーションを維持 |

- フラグなしで実行すると `-i` と同じ（アイドルスリープ防止）
- `-u` は `-t` を指定しないとデフォルト5秒でアサーションが切れる
- `-i` でシステムが起きていればディスクアクセスがある限りディスクも寝ない。`-m` が要るのはシステムは起きてるけどディスクアクセスが長時間ないケース（ネットワーク中継のみなど）で、通常は `-i` だけで十分
- `-d` はディスプレイを消したくない場合だけ使う。ディスプレイがオフでも `-i` があればシステムは動き続ける

## (1) fork/exec の親子逆転

`caffeinate -i make` を実行したとき、内部で何が起きているのか？普通に考えれば「caffeinate が親、make が子」と考えるんだが、実際は逆。

Apple の[ソースコード](https://github.com/apple-oss-distributions/PowerManagement/blob/main/caffeinate/caffeinate.c)にはこうコメントされている:

> Our parent might care about the total life cycle of this process, therefore rather than propagate exit status, Unix signals, Mach exceptions, etc, we just flip the normal parent/child relationship.

つまり:

1. caffeinate が `fork()` する
2. 親プロセスが `execvp()` でユーザー指定コマンド（make）に変身する
3. 子プロセス（caffeinate 自身）が親の終了を `DISPATCH_SOURCE_TYPE_PROC` + `DISPATCH_PROC_EXIT` で監視する
4. 子は `SIGINT` と `SIGQUIT` を `SIG_IGN` で無視する

この設計により、シェルから見ると `make` が直接の子プロセスになるため、終了コードやシグナルの伝搬が自然に機能する。`caffeinate` 越しでも `$?` がそのまま使える理由がこれ。よく考えられてるけど親子を逆にするという発想がなかった。

## (2) IOKit アサーション

各フラグは IOKit の電源アサーション API に1対1で対応している:

| フラグ | IOKit アサーション型 |
|--------|---------------------|
| `-i` | `kIOPMAssertionTypePreventUserIdleSystemSleep` |
| `-d` | `kIOPMAssertionTypePreventUserIdleDisplaySleep` |
| `-s` | `kIOPMAssertionTypePreventSystemSleep` |
| `-u` | `kIOPMAssertionUserIsActive` |
| `-m` | `kIOPMAssertPreventDiskIdle` |

内部的には `IOPMAssertionCreateWithDescription()` が呼ばれ、アサーション名は `"caffeinate command-line tool"` として `powerd` デーモンに登録。ポーリングや合成入力ではなく IOKit の電源管理レイヤで動作するため、CPU オーバーヘッドは実質ゼロ。つまりほとんど負荷がかからない。

プロセスがクラッシュしても IOKit がアサーションを自動解放するためゾンビも残らない。

ちなみに `pmset noidle` は caffeinate の前身で、現在は deprecated。man page には "This argument is deprecated in favor of caffeinate(8)" と明記されてる。

## 実践編

### 基本

```sh
# システムのアイドルスリープを防止（Ctrl+C で解除）
caffeinate -i

# 1時間（3600秒）だけスリープ防止
caffeinate -i -t 3600

# コマンド実行中だけスリープ防止（コマンド終了で自動解除）
caffeinate -i make build
caffeinate -i rsync -avz /src/ /dst/
```

### Makefile の SHELL 変数を書き換える

個別のコマンドに `caffeinate` を前置する代わりに、Makefile の `SHELL` 変数を書き換えると全レシピが自動的に caffeinate 経由になる

```makefile
ifeq ($(shell uname -s),Darwin)
  SHELL := caffeinate -i bash
endif

build:
	./long-running-build.sh  # 自動的に caffeinate 経由で実行される
```

Linux には `caffeinate` がないので `uname -s` などのOS分岐をいれとくと丁寧。

### .app バンドルを渡す

通常 `caffeinate -i /Applications/Slack.app/` は "Permission denied" になる、caffeinateはUnixコマンドしか引数に取れないため。

これを `open -W` 経由にする

```sh
caffeinate -i open -W -a "Slack"
```

`open -W` でアプリが終了するまで `open` 自体が終了しなくなり、アサーションが維持される。

### 既に走っているプロセスに後付けでcaffeinate

```sh
caffeinate -i -w $(pgrep make)
```


### SSH 越しにディスプレイを起こす

```sh
caffeinate -u -t 1
```

発想の逆転で、例えばSSHでMacに接続したりしたときに、Macのディスプレイを点灯させられる。長時間起こしておきたい場合は `-t` の値を大きくする。

### script で trap と組み合わせ

```sh
caffeinate -i &
CAF_PID=$!
trap "kill $CAF_PID 2>/dev/null" EXIT

# ... 長時間処理 ...
```

スクリプト終了時に caffeinate を確実に片付けられる。

### cron / launchd との組み合わせ

定期実行タスクがスリープで実行されない問題を防ぐ

```sh
# crontab 内で
* * * * * caffeinate -i -t 120 /path/to/script.sh
```

## 注意したい仕様

### バッテリー20%以下で強制解除される

バッテリーが20%を下回ると macOS はハードウェアレベルで電源アサーションを強制解放する。caffeinate でも防げない。AC 電源に繋いでいれば関係ない。

### -t / -w はコマンド指定時には無視

`caffeinate -t 3600 make` の `3600` は無視され、`make` の実行時間がアサーションの寿命になる。`-w` も同様。両立は不可ってこと。

### ゾンビプロセスが蓄積する場合がある

`caffeinate -u -t 1` を繰り返し呼ぶ実装でゾンビが蓄積した実例があるらしい。Raycast の Coffee 拡張では1時間で4852個のゾンビが発生し、`kern.maxprocperuid` に到達してシステム全体でプロセス生成不能になったとか。子プロセスの `waitpid()` が呼ばれないと起きる？らしくshellから呼ぶ分には問題ないだろうが繰り返しspawnするようなことをする場合は注意。...まぁそんなこと普通やらんだろうが。

### 複数起動

複数の `caffeinate` を同時に起動するとアサーションが重複する。害はないがたぶん無駄。いちおう `pmset -g assertions` で確認できる。

### セキュリティ：LOOBin としての caffeinate

caffeinate は [LOOBins (Living Off the Orchard Bins)](https://www.loobins.io/binaries/caffeinate/) に登録されている。LOOBins とは、macOS に標準搭載されているバイナリのうち攻撃者が悪用可能なものを体系化したプロジェクトで、Linux における LOLBAS / GTFOBins の macOS 版にあたる。

攻撃シナリオ:

```sh
caffeinate -i /tmp/payload
```

攻撃者はスリープによる中断を防ぎつつペイロードを実行できる。正規のプロセスとして動作するため EDR での検出が難しく、MITRE ATT&CK では Execution / Defense Evasion にマッピングされている。

防御側の視点では、前述の `pmset -g assertions` で不審なアサーションがないか確認するだとか、あるいは `caffeinate` の子プロセスを監視するとかが対策になりそう。

### クラムシェル

`caffeinate` はクラムシェルスリープを防止できないらしい。蓋を閉じつつ `caffeinate` したい場合は以下の条件を満たす必要がある

- 外部ディスプレイ接続
- AC 電源接続
- 外部キーボード / マウス接続

`sudo pmset -a disablesleep 1` を叩けば強制的にスリープを無効化できるが、蓋を閉じたままカバンに入れると発熱するだとかバッテリー消耗とかリスクを伴う。たぶんやらないほうがいい。

### デバッグとか

```sh
# 現在の電源設定を確認
pmset -g

# caffeinate のアサーション状態を確認
pmset -g assertions

# アサーション作成・解放のログを確認
pmset -g assertionslog

# 全て落とす
killall caffeinate
```

## おわり

SHELL 書き換えとか trap は普通に使える。
