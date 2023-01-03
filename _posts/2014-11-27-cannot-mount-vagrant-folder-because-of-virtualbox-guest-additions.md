---
layout: post
title: "/vagrantにマウントできない とか、guest additions バージョン違い とか"
date: 2014-11-27 15:09:08 +0900
comments: true
categories: Vagrant
published: true
---

だいたい`/vagrant`にマウントできなくて困るのが事の発端。

根本的な原因は Virtualbox Guest Additions をバージョン指定してインストールないし設定ができないことにあると思う。ずれているのはわかってるんだから。

別にVBoxに詳しくなりたいわけではないので、対応方針としては手っ取り早くVagrantとVirtualboxの最新版をインストールするようにした。

特にVagrantは随時新しいバージョンがリリースされるので。

+ `vagrant halt` (or `vagrant destroy`)
+ Vagrantの最新版をインストール
+ VirtualBoxの最新版をインストール
+ Vagrantfileに`configvbguestauto_update false`と書いてるなら消してよい
+ `vagrant plugin uninstall vagrant-vbguest` して一旦消す
+ `vagrant plugin install vagrant-vbguest` vbguest入れなおす
+ `vagrant up`
+ `vagrant ssh`
+ `sudo /etc/init.d/vboxadd setup;exit`
+ `vagrant reload`
