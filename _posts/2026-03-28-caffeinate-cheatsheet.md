---
layout: post
title: "macOSのcaffeinateを使い倒す -- fork親子逆転の仕組みからMakefile統合、LOOBinまで"
date: 2026-03-28 01:00:00 +0900
categories: [Developer Tools]
published: true
description: "caffeinateの内部実装（fork親子逆転、IOKitアサーション）からMakefile SHELL書き換え、trap連携、LOOBinとしてのセキュリティリスクまで。macOSの電源管理を掘り下げる。"
tags:
  - macos
  - cli-tools
  - shell
---

いろいろ工夫のしがいがあって結構おもしろいです。

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
- `-u` は `-t` を指定しないとデフォルト5秒でアサーションが切れます
- `-i` でシステムが起きていればディスクアクセスがある限りディスクも寝ません。`-m` が要るのはシステムは起きてるけどディスクアクセスが長時間ないケース（ネットワーク中継のみなど）で、通常は `-i` だけで十分です
- `-d` はディスプレイを消したくない場合だけ使います。ディスプレイがオフでも `-i` があればシステムは動き続けます

## (1) fork/exec の親子逆転

`caffeinate -i make` を実行したとき、内部で何が起きているのか？普通に考えれば「caffeinate が親、make が子」と考えるのですが、実際は逆です。

Apple の[ソースコード](https://github.com/apple-oss-distributions/PowerManagement/blob/main/caffeinate/caffeinate.c)にはこうコメントされています:

> Our parent might care about the total life cycle of this process, therefore rather than propagate exit status, Unix signals, Mach exceptions, etc, we just flip the normal parent/child relationship.

つまり:

1. caffeinate が `fork()` します
2. 親プロセスが `execvp()` でユーザー指定コマンド（make）に変身します
3. 子プロセス（caffeinate 自身）が親の終了を `DISPATCH_SOURCE_TYPE_PROC` + `DISPATCH_PROC_EXIT` で監視します
4. 子は `SIGINT` と `SIGQUIT` を `SIG_IGN` で無視します

この設計により、シェルから見ると `make` が直接の子プロセスになるため、終了コードやシグナルの伝搬が自然に機能します。`caffeinate` 越しでも `$?` がそのまま使える理由がこれです。よく考えられています、親子を逆にするという発想がありませんでした。目からウロコです

## (2) IOKit アサーション

各フラグは IOKit の電源アサーション API に1対1で対応しています:

| フラグ | IOKit アサーション型 |
|--------|---------------------|
| `-i` | `kIOPMAssertionTypePreventUserIdleSystemSleep` |
| `-d` | `kIOPMAssertionTypePreventUserIdleDisplaySleep` |
| `-s` | `kIOPMAssertionTypePreventSystemSleep` |
| `-u` | `kIOPMAssertionUserIsActive` |
| `-m` | `kIOPMAssertPreventDiskIdle` |

内部的には `IOPMAssertionCreateWithDescription()` が呼ばれ、アサーション名は `"caffeinate command-line tool"` として `powerd` デーモンに登録されます。ポーリングや合成入力ではなく IOKit の電源管理レイヤで動作するため、CPU オーバーヘッドは実質ゼロです。つまりほとんど負荷がかかりません。

プロセスがクラッシュしても IOKit がアサーションを自動解放するためゾンビも残りません。

ちなみに `pmset noidle` は caffeinate の前身で、現在は deprecated です。man page には "This argument is deprecated in favor of caffeinate(8)" と明記されています。

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

個別のコマンドに `caffeinate` を前置する代わりに、Makefile の `SHELL` 変数を書き換えると全レシピが自動的に caffeinate 経由になります

```makefile
ifeq ($(shell uname -s),Darwin)
  SHELL := caffeinate -i bash
endif

build:
	./long-running-build.sh  # 自動的に caffeinate 経由で実行される
```

Linux には `caffeinate` がないので `uname -s` などのOS分岐をいれとくと丁寧です。

### .app バンドルを渡す

通常 `caffeinate -i /Applications/Slack.app/` は "Permission denied" になります、caffeinateはUnixコマンドしか引数に取れないためです。

これを `open -W` 経由にします

```sh
caffeinate -i open -W -a "Slack"
```

`open -W` でアプリが終了するまで `open` 自体が終了しなくなり、アサーションが維持されます。

### 既に走っているプロセスに後付けでcaffeinate

```sh
caffeinate -i -w $(pgrep make)
```


### SSH 越しにディスプレイを起こす

```sh
caffeinate -u -t 1
```

発想の逆転で、例えばSSHでMacに接続したりしたときに、Macのディスプレイを点灯させられます。長時間起こしておきたい場合は `-t` の値を大きくします。

### script で trap と組み合わせ

```sh
caffeinate -i &
CAF_PID=$!
trap "kill $CAF_PID 2>/dev/null" EXIT

# ... 長時間処理 ...
```

スクリプト終了時に caffeinate を確実に片付けられます。

### cron / launchd との組み合わせ

定期実行タスクがスリープで実行されない問題を防ぎます

```sh
# crontab 内で
* * * * * caffeinate -i -t 120 /path/to/script.sh
```

## 注意したい仕様

### バッテリー20%以下で強制解除される

バッテリーが20%を下回ると macOS はハードウェアレベルで電源アサーションを強制解放します。caffeinate でも防げません。AC 電源に繋いでいれば関係ありません。

### -t / -w はコマンド指定時には無視

`caffeinate -t 3600 make` の `3600` は無視され、`make` の実行時間がアサーションの寿命になります。`-w` も同様です。両立は不可ということです。

### 複数起動

複数の `caffeinate` を同時に起動するとアサーションが重複します。害はないですがたぶん無駄です。いちおう `pmset -g assertions` で確認できます。

### セキュリティ：LOOBin としての caffeinate

caffeinate は [LOOBins (Living Off the Orchard Bins)](https://www.loobins.io/binaries/caffeinate/) に登録されています。LOOBins とは、macOS に標準搭載されているバイナリのうち攻撃者が悪用可能なものを体系化したプロジェクトで、Linux における LOLBAS / GTFOBins の macOS 版にあたります。

攻撃シナリオ:

```sh
caffeinate -i /tmp/payload
```

攻撃者はスリープによる中断を防ぎつつペイロードを実行できます。正規のプロセスとして動作するため EDR での検出が難しく、MITRE ATT&CK では Execution / Defense Evasion にマッピングされています。

防御側の検知手段ですが、

まず macOS Ventura 以降なら `eslogger` でbinaryの実行を捕捉できるようです

```sh
sudo eslogger exec | jq 'select(.process.executable.path == "/usr/bin/caffeinate")'
```

追加インストール不要ですが、実行元のアプリ（Terminal.app や iTerm2 など）に「システム設定 > プライバシーとセキュリティ > フルディスクアクセス」の許可が必要です。launchd デーモンとして常駐させる場合も同様に TCC 権限を付与します。

より軽量な方法だと前述の `pmset -g assertions` を定期実行して既知のプロセス以外がアサーションを保持していないかチェックします

```sh
if pmset -g assertions | grep -q 'caffeinate'; then
  pmset -g assertions | logger -t caffeinate-audit
fi
```

あとは osquery とか。`processes` テーブルでの親子関係を監査したり、[Santa](https://github.com/google/santa) で caffeinate 経由の実行バイナリを MONITOR モードで記録するとかもありますが、、めんどいかもしれません。いつの時代も防御側は苦しいです。

### クラムシェル

外部デバイスなしでは、 `caffeinate` はクラムシェルスリープを防止できないようです。蓋を閉じつつ `caffeinate` したい場合は以下の条件を満たす必要があります

- 外部ディスプレイ接続
- AC 電源接続
- 外部キーボード / マウス接続

`sudo pmset -a disablesleep 1` を叩けば強制的にスリープを無効化できるそうですが、蓋を閉じたままカバンに入れると発熱するだとか戻すのを忘れてバッテリー消耗とかリスクを伴います。やらないほうがよいです。

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

SHELL 書き換えや trap 連携は実務で普通に使えるやつでした。

関連記事:
- [dotfiles再構築 in 2026](/blog/dotfiles-2026/) -- Makefile やシェル周りの構成管理
- [2025版 ターミナル引きこもり生活を支えてくれる道具たち](/blog/tools-that-support-terminal-life-2025/) -- CLI ツールまとめ
