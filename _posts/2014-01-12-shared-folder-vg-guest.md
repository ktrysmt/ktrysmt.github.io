---
layout: post
title: "Vagrantで共有フォルダをマウントできなくなってしまった問題"
date: 2014-01-12 15:09:08 +0900
comments: true
categories: Vagrant
published: true
---

結論を言ってしまうと、Vagrantをバージョンアップするなら合わせてVirtualBoxも最新のものを再インストールしたほうがトラブルが少ないということ。

どうもvagrantを1.3.5に、vboxを4.3にあげたことが良くなかったらしい。vboxは4.2で安定するようだ。

いくつかの解決手段があるようなのでまとめる。

## 1.vg-guestの再インストール

`vagrant up`したあとにvbguestを実行。

```
$ vagrant vbguest --do install
GuestAdditions 4.3.6 running --- OK.
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
* base: ftp.iij.ad.jp
* extras: ftp.iij.ad.jp
* updates: ftp.iij.ad.jp
Setting up Install Process
Package gcc-4.4.7-4.el6.x86_64 already installed and latest version
Package 1:make-3.81-20.el6.x86_64 already installed and latest version
Package 4:perl-5.10.1-136.el6.x86_64 already installed and latest version
Nothing to do
Copy iso file C:\Program Files\Oracle\VirtualBox\VBoxGuestAdditions.iso into the box /tmp/VBoxGuestAdditions.iso
Forcing installation of Virtualbox Guest Additions 4.3.6 - guest version is 4.3.6
Verifying archive integrity... All good.
Uncompressing VirtualBox 4.3.6 Guest Additions for Linux............
VirtualBox Guest Additions installer
Removing installed version 4.3.6 of VirtualBox Guest Additions...
Copying additional installer modules ...
Installing additional modules ...
Removing existing VirtualBox DKMS kernel modules[  OK  ]
Removing existing VirtualBox non-DKMS kernel modules[  OK  ]
Building the VirtualBox Guest Additions kernel modules[  OK  ]
Doing non-kernel setup of the Guest Additions[  OK  ]
You should restart your guest to make sure the new modules are actually used

Installing the Window System drivers[FAILED]
(Could not find the X.Org or XFree86 Window System.)
An error occurred during installation of VirtualBox Guest Additions 4.3.6. Some functionality may not work as intended.
In most cases it is OK that the "Window System drivers" installation failed.
```

## 2.vboxの再セットアップ

`vagrant ssh`のあとにvboxのセットアップを実行する。
その後、`vagrant halt && vagrant up`を実行する。

```bash
$ vagrant ssh
$ sudo /etc/init.d/vboxadd setup
Removing existing VirtualBox DKMS kernel modules           [  OK  ]
Removing existing VirtualBox non-DKMS kernel modules       [  OK  ]
Building the VirtualBox Guest Additions kernel modules
Building the main Guest Additions module                   [  OK  ]
Building the shared folder support module                  [  OK  ]
Building the OpenGL support module                         [FAILED]
(Look at /var/log/vboxadd-install.log to find out what went wrong)
Doing non-kernel setup of the Guest Additions              [  OK  ]
```

```bash
$ vagrant halt
$ vagrant up
```

[参考:http://blog.mizoshiri.com/archives/1390](http://blog.mizoshiri.com/archives/1390)

## 3.ネットワークエラーが出てマウントできない

ネットワーク系のエラーが出ているときはこれで解決するようだ。
場所を特定するならgrepやackで探すといい。ただし、vagrantのバージョンによって上記のディレクトリ構成は異なるようなので注意。以下はWindows 7 + Cygwinでのコマンド。
普段はCentOSを使うので、今回はredhatフォルダ以下のファイルを編集する。

### 3-1.ホスト側、vagrantのソースのネットワークエラーチェックを外す

vagrant 1.3.5の場合は以下。

```bash
$ cd /cygdrive/c/HashiCorp/Vagrant #Vagrantがインストールされているフォルダへ移動
```

ファイル検索。

```bash
$ grep -lr "ifup eth" *
embedded/gems/gems/vagrant-1.3.5/plugins/guests/debian/cap/change_host_name.rb
embedded/gems/gems/vagrant-1.3.5/plugins/guests/debian/cap/configure_networks.rb
embedded/gems/gems/vagrant-1.3.5/plugins/guests/redhat/cap/configure_networks.rb
embedded/gems/gems/vagrant-1.3.5/plugins/guests/suse/cap/configure_networks.rb
^C
```

ifup eth の error_check を false に変更。

```bash
$ vi embedded/gems/gems/vagrant-1.3.5/plugins/guests/redhat/cap/configure_networks.rb
---

>>>
machine.communicate.sudo("ARPCHECK=no /sbin/ifup eth#{interface} 2> /dev/null")
<<<
machine.communicate.sudo("ARPCHECK=no /sbin/ifup eth#{interface} 2> /dev/null", :error_check => false)
```

### 3-2.Vagrant起動時のDHCP自動起動をOFFにする

ファイルを検索。

```bash
$ grep -lr ONBOOT *
embedded/gems/gems/vagrant-1.3.5/templates/guests/fedora/network_dhcp.erb
embedded/gems/gems/vagrant-1.3.5/templates/guests/fedora/network_static.erb
embedded/gems/gems/vagrant-1.3.5/templates/guests/redhat/network_dhcp.erb
embedded/gems/gems/vagrant-1.3.5/templates/guests/suse/network_dhcp.erb
^C
```

ONBOOTをnoに変更。

```bash
$ vi embedded/gems/gems/vagrant-1.3.5/templates/guests/redhat/network_dhcp.erb
----
>>>>
ONBOOT=yes
<<<<
ONBOOT=no
```

[参考：http://blog.monochromegane.com/blog/2013/04/06/vagrant-network-error/](http://blog.monochromegane.com/blog/2013/04/06/vagrant-network-error/)
