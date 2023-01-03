---
layout: post
title: "add 3rd repositories to centos 6"
date: 2015-06-29 15:09:08 +0900
comments: true
categories: CentOS
published: true
---

もう何度も使ってるがいい加減コピペでほしくなるのでそれ用に立てることに。

```
cd
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
rpm -Uvh remi-release-6.rpm
rpm -Uvh rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
```
