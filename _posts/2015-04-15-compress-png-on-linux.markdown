---
layout: post
title: "CentOSでadvpngを使ってpngを圧縮する"
date: 2015-04-10 15:09:08 +0900 
comments: true
categories: CentOS
published: true
---

PNGに限るやり方っぽいが割と使いやすくおすすめ、軽くて早い。

## Install

```
yum -y install zlib-devel gcc
wget http://prdownloads.sourceforge.net/advancemame/advancecomp-1.19.tar.gz
tar zxvf advancecomp-1.19.tar.gz
cd advancecomp-1.19
./configure && make && make install
```

## Usage

avzpng -z で圧縮上書き。

```
advpng -z hogehoge.png
```

jpgのやり方も抑えておくと広範に使えていいかも。

## おまけ

lossypngのほうが圧縮率は高いかも。次機会あったら使い方まとめる。

- <https://github.com/foobaz/lossypng>
