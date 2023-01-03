---
layout: post
title: "sedで正規表現するなら拡張正規表現"
date: 2015-03-21 15:09:08 +0900
comments: true
categories: Linux
published: true
---

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
報告の際、“Subject:”フィールドのどこかに“sed”を入れてください。
```

CentOSを使っている場合はgnu版が入るようだ。`-r`を使うと拡張正規表現が使える。

基本正規表現と拡張正規表現の違いは<http://d.hatena.ne.jp/entree/20141126/1417016871>に詳しく載ってる。

普段意識しないところでもエスケープが必要になってくるのは時間がもったいないというか混乱のもとなので、正規表現使いたいならば`-r`を入れた方がいい。


冒頭の置換コマンドで言うとほかポイントとしては、

- デリミタはスラッシュでなくてもいい。例では`#`を使ってる
- グルーピングされた内容を後方参照する場合は、Perl等だと`$1`だがsedは`\1`を使う。

