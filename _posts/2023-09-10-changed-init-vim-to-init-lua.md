---
layout: post
title: "vimrcをluaで書き換えてlazy.nvimいれて、起動時間30ms切りを達成した"
date: 2023-09-10 21:30:00 +0900
categories: Vim
published: true
use_toc: false
description: ""
---

luaに書き換えるのがだるくて移行を見送っていたのですが、[lazy.nvim](https://github.com/folke/lazy.nvim)の遅延読み込みが非常に優れているとのことで最近頑張って書き換えました。

## 計測

元は vimL + vim-jetpack の構成で、vim-startuptimeはだいたい100msくらいでした。

```
Total Average: 101.204100 msec
Total Max:     110.037000 msec
Total Min:      97.406000 msec
```

これを lua + lazy.nvimに置き換え、遅延読み込みを工夫した結果、

```
Total Average: 22.251000 msec
Total Max:     26.319000 msec
Total Min:     19.309000 msec
```

平均25~26ms、30msは確実に下回るくらいまで改善することができました。

やる前からすでに100msで十分に早いとも感じて、果たしてやる意味はあるのか？と正直疑問でした。

ですが100msと25msは体感でも結構違いがわかります。

起動時間が文字通り一瞬でcat/bat/lessあたりと遜色ないです。

## lazy.nvimについて

lazy.nvimの遅延読み込みは

1. cmd
2. keys
3. ft
4. event

の４つがありますが、特にeventの種類が豊富なのがいいです。

1. cmd,keys,ft で済むならそちらを設定
2. event は極力 InsertEnter, BufWritePost などユーザー操作による発火で対応
3. 2.でだめなものは VeryLazy を設定
4. 1.2.3.ではうまく動かないものは諦めて BufReadPre などを設定

LSPだけはBufReadPre,BufNewFileで設定してあげないとうまく動かなかったのですが、それがなければおそらく平均20ms程度になると思います。

ただ一点だけ副作用があり `:source $MYVIMRC` が効かないことが多いです。
なので設定変更を行った際にはnvimを一度落として上げ直したりしてます。


## lua化について

意外と覚えることは少なく、vimLに比べ書きやすいのでやってよかったです。
`vim.cmd` でvimLそのまま投げられるというのも移行のハードルが低く嬉しいです。
やや複雑なexprなmapなどは下手にlua化するとコードが肥大化しやすいので、その辺を `vim.cmd` で妥協しました。
今回はやりませんでしたが「プラグイン管理にはlazy.nvim使いつつ設定は全部 `vim.cmd` で投げ込む」というのも、乱暴ですが移行初期にはアリかなと思います。

## そもそも設定がめんどい人は

[LazyVim](https://www.lazyvim.org/)とか[AstroNvim](https://astronvim.com/)とか入れればいいと思います。これらもlazy.nvimが使われてるので起動も速くUIがリッチです。
ちなみに手元の環境でAstroNvimいれたnvimのvim-startuptimeは平均40msくらいでした。
