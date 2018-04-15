---
layout: post
title: "Cygwinにapt-cygをインストールする"
date: 2014-02-02 15:09:08 +0900 
comments: true
categories: Windows
published: true
---

手順がまとまっていなかったので自分用にメモ。
cygwin上で以下のコマンドを実行すれば良い。

ちなみにWindows7です。

```
wget http://apt-cyg.googlecode.com/svn/trunk/apt-cyg
mv apt-cyg /usr/bin
chmod +x /usr/bin/apt-cyg
```
