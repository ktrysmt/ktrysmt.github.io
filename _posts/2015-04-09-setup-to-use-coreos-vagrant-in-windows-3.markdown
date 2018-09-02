---
layout: post
title: "Windowsでcoreos-vagrant,etcd,fleetを使う(3)"
date: 2015-04-09 15:09:08 +0900
comments: true
categories: CoreOS
published: true
---



前回（<http://keidrip.github.io/blog/setup-to-use-coreos-vagrant-in-windows-2/>）の続き。

## fleet

Windowsではfleetctlが使えないらしい。IssueはOpenなようだが。

しかたがないのでVagrantで別にLinuxを立ち上げ、そっちにfleetctlを入れてみる。

### use vagrant

普段CentOSをよく使うのでなんとなくそれで。

```
$ mkdir -p /path/to/centos6
$ cd /path/to/centos6
$ vagrant init chef/centos-6.5
$ vagrant up
$ vagraht ssh
```

### install golang

<https://golang.org/dl/>から目的のソースをダウンロード。
なるべく新しいもののほうがいいと思われる。
今回は<https://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz>を選択。


### install fleetctl

無事にCentOSにログインできたら以下を実行。バージョン番号が確認できればOK。

```
$ sudo -s
# cd ~/
# wget https://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz --no-check-certificate
# tar -C /usr/local -xzf go1.4.2.linux-amd64.tar.gz
# touch ~/.bashrc
# echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
# source ~/.bashrc
# yum -y install git
# git clone https://github.com/coreos/fleet
# cd fleet
# ./build
# cp bin/fleetctl /usr/local/bin/
# fleetctl --version
fleetctl version 0.9.2+git
```

ひとまずCentOSからCoreOSにpingを打って疎通を確認する。
ゲートウェイIPと照らし合わせIPがBかCブロックまで同じならどのCoreOSマシンにもpingは通るはず。

ためしに２号機にpingしてみる。

```
# ping 192.168.66.102
PING 192.168.66.102 (192.168.66.102) 56(84) bytes of data.
64 bytes from 192.168.66.102: icmp_seq=1 ttl=64 time=1.30 ms
64 bytes from 192.168.66.102: icmp_seq=2 ttl=64 time=0.531 ms
```

確認できたら、`fleetctl --endpoint http://192.168.66.102:4001 list-machines -l` を実行してcoreos内部で実行した結果と同じ結果が取得できればOK。

環境変数をセットして楽をする。

```
$ export FLEET_ENDPOINT="http://192.168.66.102:4001"
$ fleetctl list-machines -l
```

ここまでは<http://qiita.com/voluntas/items/fc5b992fc3a579029566> とだいたい同じ。

トンネルするやり方だけ少し違う。

まずWindowsローカルに戻って、coreos-vagrantのVagrantfileがあるフォルダに移動。
その後鍵の場所を確認してそれをCentOSのVagrantfileがある場所にコピー。
コピーした鍵をCentOS内で使う。
ポート番号は22のデフォルトでOK。
CoreOS郡とCentOSは、この場合VirtualhostのゲートウェイIPとCブロックまで一緒にしてるので、同じネットワーク内にあるように振る舞ってくれる？という理解であってるかな。つまりポートフォワーディングする必要はない。

最初にpingしたのと同じ２号機(core-02)をそのまま使うことにする。

```
$ cd coreos-vagrant
$ cp $(vagrant ssh-config core-02 | sed -n "s/IdentityFile//gp") /path/to/centos6/
```

その後CentOS内で以下を実行。
Vagrantの鍵名はデフォルトではinsecure_private_keyだと思うのでそれをssh-add。

```
# eval `ssh-agent`
Agent pid 29342
# ssh-add /vagrant/insecure_private_key
Identity added: /vagrant/insecure_private_key (/vagrant/insecure_private_key)
# export FLEETCTL_TUNNEL="192.168.66.102"
# echo $FLEETCTL_TUNNEL
192.168.66.102
```

もう一度fleetctlを操作してみる。

```
# fleetctl list-machines -l
```

fleet unitを使ってサービスの起動やサービス内へのsshなどを体験。
fleetctl経由で人間がサーバを指定しなくとも勝手に適当なCoreOS内でサービスが一個起動するんだが、目の当たりにした時はなんというかちょっと感動した。

```
# fleetctl list-units
UNIT    MACHINE ACTIVE  SUB
# fleetctl submit ~/fleet/examples/hello.service
# fleetctl list-units
# fleetctl start hello.service
Unit hello.service launched on 096ba1be.../192.168.66.102
# fleetctl list-units
UNIT            MACHINE                         ACTIVE  SUB
hello.service   096ba1be.../192.168.66.102      active  running
# fleetctl status hello.service
   hello.service - Hello World
   Loaded: loaded (/run/fleet/units/hello.service; linked-runtime; vendor preset: disabled)
   Active: active (running) since Wed 2015-04-08 14:06:08 UTC; 1min 30s ago
 Main PID: 1743 (bash)
   CGroup: /system.slice/hello.service
           ├─1743 /bin/bash -c while true; do echo "Hello, world"; sleep 1; done
           └─1936 sleep 1

Apr 08 14:07:29 core-02 bash[1743]: Hello, world
Apr 08 14:07:30 core-02 bash[1743]: Hello, world
Apr 08 14:07:31 core-02 bash[1743]: Hello, world
Apr 08 14:07:32 core-02 bash[1743]: Hello, world
Apr 08 14:07:33 core-02 bash[1743]: Hello, world
Apr 08 14:07:34 core-02 bash[1743]: Hello, world
Apr 08 14:07:35 core-02 bash[1743]: Hello, world
Apr 08 14:07:36 core-02 bash[1743]: Hello, world
Apr 08 14:07:37 core-02 bash[1743]: Hello, world
Apr 08 14:07:38 core-02 bash[1743]: Hello, world
# fleetctl ssh hello.service
Last login: Wed Apr  8 14:07:38 2015 from 192.168.66.102
CoreOS stable (607.0.0)
core@core-02 ~ $
core@core-02 ~ $ ps aux |grep "Hello, world" | grep -v grep
root      1997  0.0  0.2  14052  3000 ?        Ss   14:08   0:00 /bin/bash -c while true; do echo "Hello, world"; sleep 1; done
```

あとはlist-units使ったりしてfleetを体験してみてください。

ただフェイルオーバーについて、サービスがVMをまたぐときにちょっと時間がかかった気がした。1分もかからなかったけど、数十秒ほどフェイルオーバーが完了するまで待たされた感じだった。

いきなりCoreOSをhaltしたから仕方ないのかな？いやでも突然ホストマシンが落ちることもあるので、そういった自体になってもサービスが落ちないようにしたいんだが、そこまでは無理なのかな。

プロダクションで使うときはどうするのだろう。nginx-proxyなんかを使うんだろうか。

このままではプロダクションで使えない気がするので、もう少し調べとく必要がありそう。

最後に、coreos-vagrantを上げたり下げたりしてると都度`ssh-keygen -R`てきなことをしないといけないのでコピペ用にメモ。
bashrcやzshrcに貼っとくのがいいかも。

```
eval `ssh-agent`
ssh-keygen -R 192.168.66.101
echo "" > ~/.fleetctl/known_hosts
echo "" > ~/fleet/fixtures/known_hosts
ssh-add /vagrant/insecure_private_key
export FLEETCTL_ENDPOINT="http://192.168.66.101:4001"
export FLEETCTL_TUNNEL="192.168.66.101"
fleetctl list-machines
```

## Reference

- <https://github.com/coreos/coreos-vagrant>
- <https://golang.org/dl/>
- <http://tsubalog.hatenablog.com/entry/how-to-solve-terminal-capability>
- <http://knowledge.sakura.ad.jp/tech/2519/>
- <http://qiita.com/snumano/items/381732bca47918f949df>
- <http://qiita.com/voluntas/items/fb4174b4327c539e1fe7>
- <http://qiita.com/voluntas/items/fc5b992fc3a579029566>
