---
layout: post
title: "bashでautojumpを使う"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Bash
published: true
---

## install 

```
$ git clone git://github.com/joelthelion/autojump.git
$ cd autojump
$ ./install.py                                                                                                        
```

## .bashrc追記

```
$ vim $HOME/.bashrc

[[ -s /root/.autojump/etc/profile.d/autojump.sh ]] && source /root/.autojump/etc/profile.d/autojump.sh
```

```
$ source $HOME/.bashrc
```

## how to use

```
$ j 文字列
```

文字列を入れたのち、TABキーを押すと今までにたどったcdの履歴からサジェストしてくれる。
