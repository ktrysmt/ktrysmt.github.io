---
layout: post
title: "xargsで並列処理したいときに使うオプションをメモ"
date: 2015-04-11 15:09:08 +0900 
comments: true
categories: Linux
published: true
---

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

