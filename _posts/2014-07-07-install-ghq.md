---
layout: post
title: "CentOS 6.5にghqをインストールする"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Git
published: true
description: "CentOS 6.5にGo言語経由でghqをインストールする手順。Gitリポジトリ管理ツールghqの導入方法を紹介"
tags:
  - ghq
  - git
  - golang
  - centos
---

## install go
参考：<http://ktrysmt.github.io/install-peco/>

## install ghq

```
$ yum -y install hg
$ go get github.com/motemen/ghq
$ which ghq
```
