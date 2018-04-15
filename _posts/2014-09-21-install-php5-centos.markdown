---
layout: post
title: "[CentOS] PHP 5.3を5.5にバージョンアップ"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: PHP
published: true
---

CentOS6 64bitです。

## uninstall PHP 5.3

```
yum list installed php* # アンインストールしたいのをメモっとく
yum remove php*
yum list installed php* # アンインストールされたか確認
```

## add 3rd repositories

install 

```
cd 
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm 
rpm -Uvh epel-release-6-8.noarch.rpm 
rpm -Uvh remi-release-6.rpm 
rpm -Uvh rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm 
```

switch

```
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/rpmforge.repo
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/remi.repo
```

## check php55

```
yum list php* --enablerepo=remi-php55
```

## install php55

remove前にメモっておいたのを入れるようにするといいが、無事故は保証できない。

```
yum install --enablerepo=remi-php55 php php-devel ... #いれたいものをいれる
```

自分がよく使うのは以下。

```
yum install --enablerepo=epel,remi-php55 php-devel php-fpm php-mysql php-mbstring
```
