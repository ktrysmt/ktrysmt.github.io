---
layout: post
title: "Windowsでcoreos-vagrant,etcd,fleetを使う(2)"
date: 2015-04-09 15:09:08 +0900
comments: true
categories: CoreOS
published: true
---

前回（<http://keidrip.github.io/blog/setup-to-use-coreos-vagrant-in-windows-1/>）の続き。




## vagrant up

いよいよ起動。その前に設定ファイルを編集。

### config.rb

```
-#$update_channel='alpha'
+$update_channel='stable'
-#$num_instances=1
+$num_instances=2
```

せっかくetcd設定したので２台同時に上げてみる。

### user-data

```
-#discovery: https://discovery.etcd.io/<token>
+discovery: https://<ゲートウェイIP>:4001/<任意の文字列>
```

立ち上げたetcdをCoreOSで使いたいので、user-dataを編集する。
さきほどetcdへ設定で使用したゲートウェイIPや鍵（任意の文字列）を使う。

### Vagrantfile

後述のWindowsローカルでfleetがビルドできない関係で、IP帯を通常使うvarantと揃えておく。

```
-ip = "172.12.8.#{i+100}"
+ip = "192.168.66.#{i+100}"
```

### coreos-vagrantを起動


```
$ vagrant up
```

`vagrant status`や`vagrant global-status`で状態確認。sshする場合はidやnameを使えばOK。

```
$ vagrant global-status
id       name    provider   state   directory
-------------------------------------------------------------------------------------
a70d87c  core-01 virtualbox running c:/path/to/dir/coreos-vagrant
49256c9  core-02 virtualbox running c:/path/to/dir/coreos-vagrant

$ vagrant ssh a70d87c # core-01にログイン
$ vagrant ssh core-02 # core-02にログイン
```

ここでWindowsの場合は注意が必要。デフォルトではCoreOS内でvimを使おうとしても画面がバグる。
Windowsローカルで使っているターミナルがmsysだったりcygwinだったりするとうまくviを扱えない様子。
なのでsshログイン後以下を実行する。CoreOS内で色々な設定ファイルを覗いたりいじったりしたいので、rootでやっておく。

```
$ sudo -s
# echo "export TERM=vt100" >> ~/.bashrc
# source ~/.bashrc
```

これでとりあえず起動してsshして中を覗くところまでは確認できた。

fleetctlでlist-machinesできるか確認。

```
core@core-01 ~ $ fleetctl list-machines
MACHINE         IP              METADATA
77758dac...     192.168.66.102    -
dacebe9e...     192.168.66.101    -
```

次回はfleetを使う。
最初Windowsのローカルでfleetをbuildしようとしたらハマってしまった。
実行バイナリがexeであればと思ったんだがないっぽいので、
苦肉の策でCoreOSとは別に更にVagrantでVMを一個起動してそのVMからfleetctlを使う。

なぜそこまでしてWindowsを使おうとするのかといわれると、特にこだわりはないんだけれど。

## Reference

- <https://github.com/coreos/coreos-vagrant>
- <https://golang.org/dl/>
- <http://tsubalog.hatenablog.com/entry/how-to-solve-terminal-capability>
- <http://knowledge.sakura.ad.jp/tech/2519/>
- <http://qiita.com/snumano/items/381732bca47918f949df>
- <http://qiita.com/voluntas/items/fb4174b4327c539e1fe7>
- <http://qiita.com/voluntas/items/fc5b992fc3a579029566>


