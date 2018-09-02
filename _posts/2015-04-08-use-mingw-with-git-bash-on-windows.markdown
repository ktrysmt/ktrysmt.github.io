---
layout: post
title: "WindowsのGit BashでMinGWを使う"
date: 2015-04-08 15:09:08 +0900
comments: true
categories: Windows
published: true
---

通常はGit BashにはMinGWで使える色々なコマンドが使えない。かといってMinGW一本化するとGitのバージョンが古かったりして困ったりもする。

両方インストールしてbinへのパス通せばいいんじゃないか？

同じコマンドがそれぞれに入ってたらどうなるのかわからないが、Git Bashにrsyncが入っていなかったりgccが入っていなくて困る問題は、一応これで解決できる。

## Install MinGW

以下から最新のmingw-get-setup.exeを落としてインストール。

- <http://sourceforge.net/projects/mingw/>

## Install Git Bash

Git for Windowsからインストール。昔に比べてちょっとおしゃれになっている。

- <https://msysgit.github.io/>

## put .bashrc

Git Bashを起動して以下を実行

```
touch ~/.bashrc
echo 'export PATH=$PATH:/c/MinGW/bin' >> ~/.bashrc
source ~/.bashrc
```

これでmingw-getとか使える。
