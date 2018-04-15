---
layout: post
title: "CentOSにGit 1.8をインストール"
date: 2014-05-24 12:48:21 +0000
comments: true
categories: Git
published: true
---

yumでのインストールだと1.7系までみたいなので、1.8は手でインストールする必要がある。

個人的に使いたいものがどれも1.8委譲を要求するので、仕方ないが入れる。

## インストール

```
# wget https://git-core.googlecode.com/files/git-1.8.5.2.tar.gz
# tar zxvf git-1.8.5.2.tar.gz
# cd git-1.8.5.2
# ./configure
# make 
# make install
```

インストールされたことを確認。

```
$ which git
/usr/local/bin/git
$ git --version
git version 1.8.2.1
```

## 参考
- [CentOS 6.4にgitの最新版をソースから入れる - /dev/null](http://gitpub.hatenablog.com/entry/2013/07/04/001010)
