---
layout: post
title: "[Bash] yumでバージョンアップできない"
date: 2014-10-01 15:09:08 +0900
comments: true
categories: Bash
published: true
---

理由としてはyum repositoryの向け先に問題があるかサーバに登録してるrepositoryにbashのアップデートが行き渡っていないか。

関連して、OSのバージョンがすごく古いのか…、などなど。

## RPMを入手する

とりあえずRPMはここから手に入る。

- <http://fr2.rpmfind.net/linux/rpm2html/search.php?query=bash&submit=Search+...>

必要なRPMを探して落とせばいい。

今回は`bash-3.2-33.el5_11.4`を使う。

## 脆弱性テストコマンド

```
$ env x='() { :;}; echo vulnerable'  bash -c "echo this is a test"
vulnerable
this is a test
```

## 取得

```
$ wget ftp://fr2.rpmfind.net/linux/centos/5.11/updates/x86_64/RPMS/bash-3.2-33.el5_11.4.x86_64.rpm
```

## RPMアップデートのテストをする

問題ない場合は特に何も出力はない。

```
$ sudo rpm -Uvh --test bash-3.2-33.el5_11.4.x86_64.rpm
準備中...                ########################################### [100%]
```

## 実行

```
$ sudo rpm -Uvh bash-3.2-33.el5_11.4.x86_64.rpm
$ sudo /sbin/ldconfig
```

## 脆弱性テストコマンドをもう一度。

```
$ env x='() { :;}; echo vulnerable'  bash -c "echo this is a test"
this is a test
```
