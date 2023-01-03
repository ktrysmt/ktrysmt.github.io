---
layout: post
title: "[CentOS] vsftpdインストールと設定まとめ"
date: 2014-09-06 15:09:08 +0900
comments: true
categories: CentOS
published: true
---

面倒なのでコピペで運用できるようにした。

## install

```
yum -y install vsftpd
```

## 設定変更

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

## 起動

```
/etc/init.d/vsftpd start
```

## 参考
FTPサーバー構築(vsftpd) - CentOSで自宅サーバー構築 <http://centossrv.com/vsftpd.shtml>
