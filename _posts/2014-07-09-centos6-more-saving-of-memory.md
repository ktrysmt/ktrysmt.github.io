---
layout: post
title: "[CentOS6] 不要なデーモンを止めてメモリを節約する"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: CentOS
published: true
---

## 不要なデーモンを止めます

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


## コンソール数を減らす

```
vim /etc/sysconfig/init

-ACTIVE_CONSOLES=/dev/tty[1-6]
+ACTIVE_CONSOLES=/dev/tty1
```

## 再起動

```
shutdown -rf +30 &
```

## 参考

- [「さくらのVPS」を使ってみる - さくらインターネット創業日記](http://tanaka.sakura.ad.jp/archives/001061.html)

- [CentOSの不要なデーモンを停止してみる - yk5656 diary](http://d.hatena.ne.jp/yk5656/20140412/1397873149)



