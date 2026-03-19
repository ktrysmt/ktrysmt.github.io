---
layout: post
title: "Linuxコマンドライン Tips集"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Linux
published: true
description: "pssh、sed、grep、xargs、rsync、wgetなどLinuxコマンドラインの実用的なTipsをまとめた記事。並列処理やMakefileヘルプの小技も紹介。"
tags:
  - linux
  - shell
  - devops
  - infrastructure
  - cli-tools
  - performance
redirect_from:
  - /blog/pssh-and-sed/
  - /blog/make-swapfile/
  - /blog/usage-grep-xargs-sed/
  - /blog/generate-static-pages-by-wget-options/
  - /blog/use-regexp-by-sed/
  - /blog/use-xargs-in-parallel/
  - /blog/rsync-option/
  - /blog/write-useful-help-command-by-shell/
---

## psshとsedの組み合わせで複数サーバを一括置換

本番環境の状態を変更するなど時代と逆行しているが、必要なときもあったりするのでメモ。

### install

```
$ wget https://parallel-ssh.googlecode.com/files/pssh-2.3.1.tar.gz
$ tar xzf pssh-2.3.1.tar.gz
$ cd pssh-2.3.1
$ python setup.py build
$ sudo python setup.py install
```

### ノードリスト作成

```
$ vim nodelist.txt
```

```
192.168.33.11
192.168.33.12
192.168.33.13
```

### 実行

```
$ sudo pssh -h nodelist.txt -i "sed -i 's/hoge/fuga/g' /path/to/file"
```

## スワップファイルの作成

たまに必要になるのでコピペ用にメモ。

### 作成

```
sudo dd if=/dev/zero of=/swapfile bs=1024K count=1024 # 1GB欲しい場合
sudo chmod 0600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### スワップの確認

```
 swapon -s
```

### /etc/fstabに追記

```
sudo bash -c 'echo "/swapfile               swap                    swap    defaults        0 0" >> /etc/fstab'
```

## grep、xargs、sedを使ったワンライナー集

### grepして出てきた文字列を置換

```
grep -lr hogehoge path/to/dir | xargs -I X sed -i "s/hogehoge/fugafuga/g" X
```

### リモートのマージ済みブランチを消す

```
git push origin $(git branch -r --merged | grep origin/ | grep -v master | sed s~origin/~:~)
```

### access_logからIPベースでアクセス数を集計

```
cat /var/log/httpd/access_log | awk '{print $1}' | sort | uniq -c
```

## Wgetで静的サイト生成

html拡張子を付ける場合

```
nohup wget --recursive --no-clobber --page-requisites --html-extension --convert-links --restrict-file-names=windows --domains ドメイン名 --no-parent http://ドメイン名/ > LOG.txt &
```

付けない場合

```
nohup wget --recursive --no-clobber --page-requisites --convert-links --restrict-file-names=windows --domains ドメイン名 --no-parent http://ドメイン名/ > LOG.txt &
```

処理ログ抜いておくのと、Windowsでも参照できるようにしておく。

ドメイン名揃えておくと同一ドメイン内のみのリンク（href）だけ辿ってくれる。

えてしてこの手の処理は時間がかかることが多いのでnohupとバックグラウンド処理を付けておく。

## sedで拡張正規表現を使う

たまに使う置換コマンド。

```
sed -i -r 's#href="(.+)"#href="/path/to/\1"#g' filename
```

とか。これだけみてもわからないので、まずはsedのヘルプ↓

```
使用法: sed [オプション]... {スクリプト(他になければ)} [入力ファイル]...

  -n, --quiet, --silent
                 suppress automatic printing of pattern space
  -e スクリプト, --expression=スクリプト
                 実行するコマンドとしてスクリプトを追加
  -f スクリプト・ファイル, --file=スクリプト・ファイル
                 実行するコマンドとしてスクリプト・ファイルの内容を追加
  -i[接尾辞], --in-place[=接尾辞]
                 ファイルをその場で編集 (拡張子があれば、バックアップを作成)
  -c, --copy
                 use copy instead of rename when shuffling files in -i mode
                 (avoids change of input file ownership)
  -l N, --line-length=N
                 「l」コマンド用の行折返し長を指定
  --posix
                 GNU拡張を全部禁止。
  -r, --regexp-extended
                 スクリプトで拡張正規表現を使用。
  -s, --separate
                 ファイルを一連の入力にせず、別々に処理。
  -u, --unbuffered
                 入力ファイルから極小のデータを取り込み、
                 ちょくちょく出力バッファーに掃出し
      --help     この説明を表示して終了
      --version  バージョン情報を表示して終了

-e、--expression、-f、--fileオプションのどれもないと、オプション以外の
最初の引数をsedスクリプトとして解釈します。残りの引数は全部、入力ファ
イル名となります。入力ファイルの指定がないと、標準入力を読み込みます。

電子メールによるバグ報告の宛先: bonzini@gnu.org
報告の際、"Subject:"フィールドのどこかに"sed"を入れてください。
```

CentOSを使っている場合はgnu版が入るようだ。`-r`を使うと拡張正規表現が使える。

基本正規表現と拡張正規表現の違いは<http://d.hatena.ne.jp/entree/20141126/1417016871>に詳しく載ってる。

普段意識しないところでもエスケープが必要になってくるのは時間がもったいないというか混乱のもとなので、正規表現使いたいならば`-r`を入れた方がいい。

冒頭の置換コマンドで言うとほかポイントとしては、

- デリミタはスラッシュでなくてもいい。例では`#`を使ってる
- グルーピングされた内容を後方参照する場合は、Perl等だと`$1`だがsedは`\1`を使う。

## xargsで並列処理する

xargsの並列処理。@tagomoris氏の記事が参考になる。

- <http://tagomoris.hatenablog.com/entry/20110513/1305267021>

特に以下の項目。

> ただし注意点。xargsで起動されるコマンドが軽く、かつ xargs 経由で食わせる引数の数が膨大なものとなる場合、並列実行によるメリットよりもコマンド起動回数(fork回数)によるデメリットの方が上回る可能性が高い。そういう場合には -L で指定する数を多くするなどして対処しよう。

こういうコマンドを使いたいときは往々にしていて数が膨大で時間がないような事が多い。さくっとやるなら以下のように。

```
find . -not -name '*.bz2' | xargs -L 50 -P 10 bzip2
```

xargsで呼ぶコマンドの引数が１つしか受け付けないようなら、
`-L`で絞ればいい。

```
find . -not -name '*.php' | xargs -L 1 -P 4 php-cs-fixer --level=psr2 fix
```

## rsyncで押さえておくオプション

バックアップ、転送、ネットワークまたいで簡単に差分チェックと、便利なrsync。
最近あんま使わなくなってきたけどやっぱりまだたまに使うのでメモ。

### 権限まわりを上書きしない

```
rsync -avz --no-o --no-p --no-g
```

転送先になんらか制約があるときに。

### Dry-Run

```
rsync -avzn src/ dst/
```

`n`つけるだけでいいので楽。かなり便利

### 転送先のファイルの更新日時が転送元より未来だったら上書きしない

```
rsync -avzu src/ dst/
```

Work In Progress な状況でもファイルを送りつけたいときに。

### 鍵変更

```
rsync -avz -e "ssh i /path/to/key" src/ hostname:/dst/
```

普通のsshとちょっと書き方が違うので注意。

### その他

`exclude`や`include`は普通すぎるので割愛。

`avz`は基本つけときゃ間違いないが、cronで使ったり標準出力捨てるならvは取っていい。

## Makefileでヘルプコマンドを書くときの小技

自分が使うためのコピペ用ですが、せっかくなので解説もしようかと思います。

コードは以下です。

```Makefile
.DEFAULT_GOAL := help

help: ## print this message
	@echo "Example operations by makefile."
	@echo ""
	@echo "Usage: make SUB_COMMAND argument_name=argument_value"
	@echo ""
	@echo "Command list:"
	@echo ""
	@printf "\033[36m%-30s\033[0m %-50s %s\n" "[Sub command]" "[Description]" "[Example]"
	@grep -E '^[/a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | perl -pe 's%^([/a-zA-Z_-]+):.*?(##)%$$1 $$2%' | awk -F " *?## *?" '{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'
```

perl と awk で頑張っています。
最後のawk `awk -F " *?## *?" '{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'` がミソです。

これを踏まえた make target の書き方は以下のようなかんじです。

```Makefile
all: test build ## run all commands ## make all

build: ## build docker dontainer ## make build argument_name_1=argument_value_1
  @docker build .

test: ## test the node app ## make test NODE_ENV=production
  @npm test $(MAKEFLAGS)
```

` ## ` を区切り文字として、最初の区切りのあとに説明文を、次の区切りのあとにコマンド例を書きます。

そうすると、helpの出力はおよそ以下のようになります。

```sh
$ make
Example operations by makefile.

Usage: make SUB_COMMAND argument_name=argument_value

Command list:

[Sub command]        [Description]                   [Example]
all                  run all commands                make all
build                build docker dontainer          make build argument_name=argument_value
test                 test the node app               make test NODE_ENV=production
```

実際のターミナルの表示はこんな感じになります。

![](/assets/images/blog/write-useful-help-command-by-shell/makefile-stdout.png)

Sub command, Description, Example の alignment をきれいに揃えられるところが便利で気に入っています。
また Sub command の列は cyan で色付けされてちょっときれいです。

awkでやってるprintfの `'{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'` あたりの解説ですが、

色付けは `033[36m` 〜 `\033[0m` でおこなっていて、
`%-30s` の `-30` の数値で整列させる幅（スペース埋め）の調整をしています。
埋めないのならば `%s` と書けばよいです（3列目、`$$3`の項）。

地味ですが、このリポジトリなんだったっけ...？と忘れかけたときなんかに便利です、とりあえずmakeたたけば思い出せます。よければ使ってください。
