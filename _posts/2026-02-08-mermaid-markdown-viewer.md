---
layout: post
title: "ターミナルでmermaidが埋め込まれたmarkdownを見やすくするcli"
date: 2026-02-08 23:00:00 +0900
categories: Mermaid
published: true
use_toc: false
---

## つくったもの

[](https://github.com/ktrysmt/mema){:.card-preview}


## インストール

めんどいのでgithub直入れです。

```sh
npm install -g ktrysmt/mema
# Or
npm install -g git+https://github.com/ktrysmt/mema.git
```

## 使い方

```
$ mema test/test1.md
# Hello

This is markdown with mermaid:

    ┌───┐     ┌───┐
    │   │     │   │
    │ A ├────►│ B │
    │   │     │   │
    └───┘     └───┘

More text.
```

stdinでも可

```
$ echo '# Hello\n\n```mermaid\nflowchart LR\n    A --> B\n```' | mema
# Hello

    ┌───┐     ┌───┐
    │   │     │   │
    │ A ├────►│ B │
    │   │     │   │
    └───┘     └───┘
```

## おわり

beautiful-mermaid と marked, marked-temrminal を使わせてもらってます。
beautiful-mermaid が gantt に未対応なのでそこだけ出力しませんが、それ以外はかなりきれいな ascii art として出してくれます。

svg出力でもいいのですがterminal環境での画像表示は色々とストレスが多い領域なので作業中気軽にmermaidを視覚的に確認できる ascii art出力としました。
github上でviewするときと体験が変わらないのが、脳みそ邪魔しなくていい感じです。


