---
layout: post
title: "CentOSにtigをインストール"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Git
published: true
---

## ソースからインストール

```
cd /tmp
git clone git://github.com/jonas/tig.git
cd tig/
make prefix=/usr
make install prefix=/usr
```

## パッケージからインストール

CentOS6ならパッケージを使うほうがよい。

rpmforgeを使う。

```
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
yum list | grep tig | grep rpmforge # 存在確認
yum install -y tig --enablerepo=rpmforge # インストール
tig --version # インストール確認
```
