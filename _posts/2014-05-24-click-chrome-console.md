---
layout: post
title: "Chrome デバッグコンソール上で要素をClickする"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: [Programming]
published: true
description: "ChromeデバッグコンソールでjQueryのtriggerを使い、複数の要素をまとめてクリックする方法"
tags:
  - javascript
  - web
---

久しぶりにブログを書きます。

```
$(".target").each(function(){
$(this).trigger("click");
});
```

まとめてクリックしたいときに。
