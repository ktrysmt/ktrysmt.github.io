---
layout: post
title: "s3fsをインストールする"
date: 2014-10-18 15:09:08 +0900
comments: true
categories: AWS
published: true
---

環境はEC2 Amazon Linuxです。

## setup fuse

### install fuse

```
cd /tmp
wget http://sourceforge.net/projects/fuse/files/fuse-2.X/2.8.5/fuse-2.8.5.tar.gz/download
tar xvfz fuse-2.8.5.tar.gz
cd fuse-2.8.5
./configure --prefix=/usr
make
make install
```

### export and reload

```
echo "export PKG_CONFIG_PATH=/usr/lib/pkgconfig:/usr/lib64/pkgconfig/" >> ~/.bashrc
/sbin/ldconfig && source ~/.bashrc
```

## setup s3fs

### install libraries

```
yum -y install wget tar make automake gcc libstdc++-devel gcc-c++ curl-devel curl-devel libxml2-devel openssl-devel mailcap libxml-2.0 libcrypto libcurl
wget https://s3fs.googlecode.com/files/s3fs-1.74.tar.gz
tar xvzf s3fs-1.74.tar.gz
cd s3fs-1.74
./configure --prefix=/usr
make
make install
```

### setup s3 secret key

事前にAWSのS3キーとシークレットキーを取得しておく。

```
echo 'AccessKey:SecretKey' > /etc/passwd-s3fs
chmod 640 /etc/passwd-s3fs
```

### setup to mount s3

```
mkdir /mnt/s3
chmod 777 /mnt/s3
echo 'KERNEL=="fuse", MODE="0777"' > /etc/udev/rules.d/99-fuse.rules
source ~/.bashrc && modprobe fuse
```

### mounting

あらかじめs3にbucketを作っておくこと。

外部からもhttpでアクセスする場合には`allow_other`を、サーバ内で所有者およびグループを定義して運用する場合は`uid`や`gid`を引数に与えることでマウント時にその所有権限となる。

```
s3fs bucket_name /mnt/s3 -o rw,allow_other,use_cache=/tmp,uid=222,gid=500
```

### setup to auto mount

/etc/fstabに記述するやり方だと再起動時にうまくマウントしないようなので別のやり方で対応するとよい様子。

```
echo "fusermount -u /mnt/s3" >> /etc/rc.d/rc.local
echo "s3fs s3 /mnt/s3 -o allow_other -o allow_other,default_acl=public-read" >> /etc/rc.d/rc.local
```

## 参考
+ <https://code.google.com/p/s3fs/>
+ <http://angelndxp.wordpress.com/2014/03/06/%E3%80%90aws%E3%80%91-s3fs%E3%83%9E%E3%82%A6%E3%83%B3%E3%83%88%E6%99%82%E3%81%AB%E8%B5%B7%E3%81%93%E3%82%8B%E5%95%8F%E9%A1%8C%E3%81%A8%E3%80%81%E3%81%9D%E3%81%AE%E5%9B%9E%E9%81%BF%E6%96%B9%E6%B3%95/>
+ <http://d.hatena.ne.jp/kmn23/20130320/1363790903>
