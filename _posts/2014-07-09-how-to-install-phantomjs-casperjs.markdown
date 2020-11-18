---
layout: post
title: "CentOS6にphantomjs+casperjs+IPAフォントをインストール"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: NodeJS
published: true
---

## install PhantomJS

```
cd ~/
wget https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-linux-x86_64.tar.bz2
bzip2 -dc phantomjs-1.9.7-linux-x86_64.tar.bz2 | tar xvf -
ln -s ~/phantomjs-1.9.7-linux-x86_64/bin/phantomjs /usr/local/bin/phantomjs
```

## install CasperJS

```
yum -y install git
git clone git://github.com/n1k0/casperjs.git
cd casperjs
ln -sf `pwd`/bin/casperjs /usr/local/bin/casperjs
```

## LinuxにIPAフォントをインストール

```
sudo yum -y install mkfontdir mkfontscale
wget "http://sourceforge.jp/frs/redir.php?m=osdn&f=%2Fmix-mplus-ipa%2F59021%2Fmigmix-medium-20130702.tar.xz"
tar Jxf redir.php\?m\=osdn\&f\=%2Fmix-mplus-ipa%2F59021%2Fmigmix-medium-20130702.tar.xz
cd migmix-medium-20130702/
mkdir -p /usr/share/fonts/ipa/TrueType
cp migmix-* /usr/share/fonts/ipa/TrueType/
mkfontdir
mkfontscale
```
