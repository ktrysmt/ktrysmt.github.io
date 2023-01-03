---
layout: post
title: "Chrome デバッグコンソール上で要素をClickする"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: JavaScript
published: true
---

久しぶりにブログを書きます。

```
$(".target").each(function(){
$(this).trigger("click");
});
```

まとめてクリックしたいときに。
