---
layout: post
title: rg(ripgrep)でinclude/excludeする
date: 2018-04-30
categories: Rust
published: true
use_toc: false
---

### コード

```
# include
rg foo -g '*.min.js'

# exclude
rg foo -g '!*.min.js'
```

`!`を使うとincludeがexcludeに切り替わる。`-g`は`--glob`のショートハンド。
急いで探しものをしているときとか、非常に助かります。

### ちょっとした注意点

がいくつかありまして。

* `!`(exclude)とクォートを組み合わせる時はシングルクォーテーションにする
    * ダブルクオーテーションだとヒストリ展開されてしまう
* globの対象はファイルパス
    * ファイルの中身が対象ではないので注意

globしつつファイルの中身に対して絞り込み等をしたい場合は、最初の例のようにオプションなし引数で単語を渡すか、あるいは下記のように正規表現のオプション`-e`が使えます。

```
rg -e '^foo' -g '*.md'
```

### 参考
* https://qiita.com/anqooqie/items/785f46a8cc5f10ba7abb
