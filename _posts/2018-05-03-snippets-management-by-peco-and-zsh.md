---
layout: post
title: "pecoとzshでかんたんsnippet管理"
date: 2018-05-03
comments: true
categories: Zsh
published: true
use_toc: false
description: "（pecoとzshを使っている人にとっては）かんたんです。"
---

### 手順

たまに使うコマンドなどをあらかじめ`~/.snippets`に一行ずつ書いておきます。

```sh
cut -d' ' -f1,3
dig -x 1.1.1.1
tcpdump -nn src host 1.1.1.1 and port not 22
find . -type f -regextype posix-basic -regex ".*.(log|txt)" ! -regex ".*./foo/bar/.*"
:
:
```

その上で`~/.zshrc`に以下を追加。

```sh
: "peco snippet" && {
  function peco-select-snippet() {
    BUFFER=$(cat ~/.snippets | peco)
    CURSOR=$#BUFFER
    zle -R -c
  }
  zle -N peco-select-snippet
  bindkey '^x^r' peco-select-snippet
}
```

私は`<C-X><C-R>`にスニペット呼び出しを割り当ててます。

### 所感

はじめはプラグインやCLIの導入を検討してたんですが、あまり高機能にしたところで使いこなせるイメージもなく（オプションなど覚えていられないし）、ニーズもシンプルだったので薄く自作しました。

私はこのスニペットファイルをdotfiles管理下において、日々育てて（？）います。便利です。
