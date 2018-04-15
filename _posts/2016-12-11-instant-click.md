---
layout: post
title: "画面遷移の体感速度をサクッと改善するInstantClick"
date: 2016-12-11 15:09:08 +0900
comments: true
categories: JavaScript
published: true
---


```
<script src="//cdn.jsdelivr.net/instantclick/3.0.1/instantclick.min.js" data-no-instant></script>
<script data-no-instant>InstantClick.init(50);</script>
```

上記のタグを`</body>`直前などに入れ込むだけですぐ使えます。

`init()` の引数で処理が走るまでのディレイをmsec単位で制御可能。  
`a`タグにマウスオーバーするだけで自動的に`href`の先のHTMLを取りに行くので、  
あまりにディレイが短すぎると大量のリクエストが発生するので注意。


静的サイトには向いてると思います。簡単に体感速度をあげられるのでオススメ。

