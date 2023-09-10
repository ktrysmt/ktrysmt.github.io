---
layout: post
title: "init.vimをinit.luaに書き換えて起動時間30ms達成"
date: 2023-09-10 21:30:00 +0900
categories: Vim
published: true
use_toc: false
description: ""
---

luaに書き換えるのがだるくてinit.vimの移行を見送っていたのですが、[lazy.nvim](https://github.com/folke/lazy.nvim)の遅延読み込みが非常に優れているとのことで最近頑張って書き換えました。

## 計測

元は vimL + vim-jetpack の構成で、vim-startuptimeはだいたい100msくらいでした。

```
Total Average: 101.204100 msec
Total Max:     110.037000 msec
Total Min:      97.406000 msec
```

これを lua + lazy.nvimに置き換え、遅延読み込みを工夫した結果、

```
Total Average: 23.951000 msec
Total Max:     26.319000 msec
Total Min:     19.309000 msec
```

平均25~26ms、30msは確実に下回るくらいまで改善することができました。

やる前からすでに100msで十分に早いとも感じて、果たしてやる意味はあるのか？と正直疑問でした。

ですが100msと25msは体感ではっきりと違いがわかります。

起動時間がcat/bat/lessと体感ほぼ同じくらいになるので、`command | vim -` や `<C-x><C-e>` を使う機会がかなり増えました。

## lazy.nvimについて

lazy.nvimの遅延読み込みは

1. cmd
2. keys
3. ft
4. event

の４つがありますが、特にeventの挙動をプラグインごとに詰めることで改善します。

1. cmd,keys,ftで済むならそれで設定する
2. event は極力 InsertEnter, BufWritePost などユーザー操作で発火
3. 2.でだめなものは VeryLazy で対処
4. 3.ではうまく動かないものは諦めて BufReadPre, VimEnter などを設定

LSPだけはBufReadPre,BufNewFileで設定してあげないとうまく動かなかったのですが、それがなければおそらく平均20ms程度になると思います。

ただ一点だけ副作用があり `:source $MYVIMRC` が効かないことがときどき起こります。
なので設定変更を行った際にはnvimを一度落とす必要がありました。


## lua化について

意外と覚えることは少なく、vimLに比べ書きやすいのでやってよかったです。
`vim.cmd` でvimLそのまま投げることもできるというのも移行がしやすくて嬉しいです。
やや複雑なexprなmapなどは下手にlua化するとコードが肥大化しやすいので、その辺は今回 `vim.cmd` で妥協しました。
今回はlspや補完をlua系プラグインに置き換えたのでやりませんでしたが、lazy.nvim使いつつ設定は全部 `vim.cmd` で投げ込む、というのも乱暴ですがアリかなと思います。

## そもそも設定がめんどい人は

[AstroNvim](https://astronvim.com/)入れればいいと思います。こちらもlazy.nvimが使われてるので起動が速く、リッチです。
手元の環境だとAstroNvimいれたnvimはvim-startuptimeの結果が平均40msくらいでした。