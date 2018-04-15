---
layout: post
title: "「Row size too large.」と叱られたのでMySQLを5.5にしてinnodb_file_format=Barracudaにして逃げた"
date: 2015-04-12 15:09:08 +0900 
comments: true
categories: MySQL
published: true
---

MySQL 5.1は捨ててしまおう。

CentOS6です。

## install mysql5.5

```
/etc/init.d/mysqld stop
yum remove mysql*
yum install http://dev.mysql.com/get/mysql-community-release-el6-5.noarch.rpm
yum-config-manager --disable mysql56-community
yum-config-manager --enable mysql55-community
yum install yum-utils
yum install mysql mysql-devel mysql-server mysql-utilities postfix crontabs
```

## edit my.cnf

```
[mysqld]
innodb_file_per_table
innodb_file_format=Barracuda
```

## start mysql

```
rm -f /var/lib/mysql/ib_logfile*
/etc/init.d/mysqld start
```

## change row format

```
ALTER TABLE table_name ROW_FORMAT=COMPRESSED;
```

## 考察

テーブル設計MySQL的に見て間違ってるのはわかっている。

だがそれをいうならそもそもVARCHAR,TEXT,BLOBが多すぎる場合にはテーブルやカラムを作れなくするべきだと思うんだ。

作れるならデータも入れられる、と開発者は思ってしまうじゃないか。
