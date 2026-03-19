---
layout: post
title: "CentOS運用・セットアップまとめ"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Linux
published: true
description: "CentOS 6の不要デーモン停止、vsftpd構築、PNG圧縮、サードパーティリポジトリ追加、パフォーマンス解析ツールなど運用ノウハウ集。"
tags:
  - centos
  - linux
  - performance
  - infrastructure
  - cli-tools
  - mysql
  - nginx
redirect_from:
  - /blog/centos6-more-saving-of-memory/
  - /blog/install-vsftpd-centos/
  - /blog/compress-png-on-linux/
  - /blog/add-3rd-repositories-to-centos-6/
  - /blog/analizing-performancerhe-tools-for-rhel/
---

## CentOS 6で不要なデーモンを止めてメモリを節約する

### 不要なデーモンを止めます

WEBサーバを立ててるだけなので不要なデーモンが多い。

```
chkconfig haldaemon off
chkconfig messagebus off
chkconfig auditd off
chkconfig blk-availability off
chkconfig iscsi off
chkconfig iscsid off
chkconfig lvm2-monitor off
chkconfig mdmonitor off
chkconfig netfs off
chkconfig postfix off
chkconfig udev-post off
chkconfig httpd off
```

nginxに移行したのでhttpdは止めました。

### コンソール数を減らす

```
vim /etc/sysconfig/init

-ACTIVE_CONSOLES=/dev/tty[1-6]
+ACTIVE_CONSOLES=/dev/tty1
```

### 再起動

```
shutdown -rf +30 &
```

### 参考

- [「さくらのVPS」を使ってみる - さくらインターネット創業日記](http://tanaka.sakura.ad.jp/archives/001061.html)

- [CentOSの不要なデーモンを停止してみる - yk5656 diary](http://d.hatena.ne.jp/yk5656/20140412/1397873149)

## CentOSにvsftpdをインストールして設定する

面倒なのでコピペで運用できるようにした。

### install

```
yum -y install vsftpd
```

### 設定変更

```
cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.org
sed -i -e "/anonymous_enable=YES/anonymous_enable=NO/" /etc/vsftpd/vsftpd.conf
sed -i -e "/xferlog_std_format=YES/xferlog_std_format=NO/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#xferlog_file/xferlog_file/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#ascii_download_enable/ascii_download_enable/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#ascii_upload_enable/ascii_upload_enable/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#ftpd_banner/ftpd_banner/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#chroot_list_enable/chroot_list_enable/" /etc/vsftpd/vsftpd.conf
sed -i -e "/#chroot_list_file/chroot_list_file/" /etc/vsftpd/vsftpd.conf
echo "chroot_local_user=YES" >> /etc/vsftpd/vsftpd.conf
sed -i -e "/#ls_recurse_enable/ls_recurse_enable/" /etc/vsftpd/vsftpd.conf
```

### 起動

```
/etc/init.d/vsftpd start
```

### 参考

FTPサーバー構築(vsftpd) - CentOSで自宅サーバー構築 <http://centossrv.com/vsftpd.shtml>

## CentOSでadvpngを使ってPNGを圧縮する

PNGに限るやり方っぽいが割と使いやすくおすすめ、軽くて早い。

### Install

```
yum -y install zlib-devel gcc
wget http://prdownloads.sourceforge.net/advancemame/advancecomp-1.19.tar.gz
tar zxvf advancecomp-1.19.tar.gz
cd advancecomp-1.19
./configure && make && make install
```

### Usage

avzpng -z で圧縮上書き。

```
advpng -z hogehoge.png
```

jpgのやり方も抑えておくと広範に使えていいかも。

### おまけ

lossypngのほうが圧縮率は高いかも。次機会あったら使い方まとめる。

- <https://github.com/foobaz/lossypng>

## CentOS 6にサードパーティリポジトリを追加する

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

## RHEL向けパフォーマンス解析ツールまとめ

よく使うツールのまとめ。

### OS系

```
yum -y install dstat    # enhanced vmstat
yum -y install iftop    # Network
yum -y install iotop    # I/O
yum -y install htop     # enhanced top
```

### MySQL系

#### pt-query-digest

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

#### innotop

MySQL InnoDB analizier.

```
yum -y install innotop
```

#### kataribe

Analize web-server's wait times for Apache, Nginx, Rack, etc.

##### For example(nginx):

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
