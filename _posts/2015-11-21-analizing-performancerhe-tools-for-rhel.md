---
layout: post
title: "RHEL向けパフォーマンス解析ツールまとめ"
date: 2015-11-21 15:09:08 +0900 
comments: true
categories: CentOS
published: true
---

よく使うツールのまとめ。

## OS系

```
yum -y install dstat    # enhanced vmstat 
yum -y install iftop    # Network
yum -y install iotop    # I/O
yum -y install htop     # enhanced top
```

## MySQL系

### pt-query-digest

Analize the slow query log.

```
yum -y localinstall https://www.percona.com/downloads/percona-toolkit/2.2.16/RPM/percona-toolkit-2.2.16-1.noarch.rpm
mysql -u user -p # login
mysql> set global slow_query_log = 1;
mysql> set global long_query_time = 3;
mysql> set global slow_query_log_time = /tmp/mysql-slow-query.log;
mysql> exit;
pt-query-digest /tmp/mysql-slow-query.log | less
/etc/init.d/mysqld restart # stop output slow query log
```

### innotop

MySQL InnoDB analizier.

```
yum -y install innotop
```

### kataribe

Analize web-server's wait times for Apache, Nginx, Rack, etc.

#### For example(nginx):

- Add below to nginx conf.

```
log_format with_time '$remote_addr - $remote_user [$time_local] '
                     '"$request" $status $body_bytes_sent '
                     '"$http_referer" "$http_user_agent" $request_time';
access_log /var/log/nginx/access.log with_time;
```

- Install kataribe

Get latest binary from [matsuu/kataribe/releases](https://github.com/matsuu/kataribe/releases).

```
yum -y install unzip
mkdir kataribe
cd !$
wget https://github.com/matsuu/kataribe/releases/download/v0.3.0/linux_amd64.zip # for RHEL (64bit)
unzip linux_amd.zip
/etc/init.d/nginx reload # apply with_time about access_log at nginx.
cat /var/log/nginx/access.log | ./kataribe | less
```
