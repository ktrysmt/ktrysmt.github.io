---
layout: post
title: "psshとsed一括置換の組み合わせ"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Linux
published: true
---

本番環境の状態を変更するなど時代と逆行しているが、必要なときもあったりするのでメモ。

## install

```
$ wget https://parallel-ssh.googlecode.com/files/pssh-2.3.1.tar.gz
$ tar xzf pssh-2.3.1.tar.gz
$ cd pssh-2.3.1
$ python setup.py build
$ sudo python setup.py install
```

## ノードリスト作成

```
$ vim nodelist.txt
```

```
192.168.33.11
192.168.33.12
192.168.33.13
```

## 実行

```
$ sudo pssh -h nodelist.txt -i "sed -i 's/hoge/fuga/g' /path/to/file"
```

