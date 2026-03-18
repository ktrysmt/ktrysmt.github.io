---
layout: post
title: "foreverでungitをデーモン化"
date: 2014-08-31 15:09:08 +0900
comments: true
categories: Git
published: true
description: "Node.jsのforeverモジュールを使ってungitをデーモン化する方法。npmでインストールしbash_profileに設定する手順を紹介"
tags:
  - git
  - node-js
  - javascript
---

デーモン書いていましたがこちらのほうがスマートな印象。
再起動も自動でしてくれるようです。

## install

nodeとnpmがインストールされているのを前提に。

```
npm -g install forever ungit
echo "forever `which ungit` > /dev/null &" >> ~/.bash_profile
```

## 反映

```
source ~/.bash_profile
```

