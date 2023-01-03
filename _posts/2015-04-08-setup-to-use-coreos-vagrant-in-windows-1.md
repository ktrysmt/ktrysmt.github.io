---
layout: post
title: "Windowsでcoreos-vagrant,etcd,fleetを使う(1)"
date: 2015-04-09 15:09:08 +0900
comments: true
categories: CoreOS
published: true
---

前回（<http://ktrysmt.github.io/blog/about-coreos-docker-golang-and-kocha/>）の続き。

Go先にやるといっときながら楽しそうなCoreOSから触っている。

## Environment

例によってWindowsを使ってるのでそれ用の設定です。

環境は以下。

- Windows7 64bit
- Vagrant 1.7.2
- Virtualbox 4.3.20
- Git Bash
- rsync (後述)

私は他にも色々入れているが最低限上記で足りると思う。

参考：<http://ktrysmt.github.io/blog/minimal-install-my-machines/>

## how to setup

Git Bashを起動してWindowsローカル上で以下を実行。

```
$ git clone --recursive https://github.com/coreos/coreos-vagrant/
$ cd coreos-vagrant
$ mv user-data.sample user-data
$ mv config.rb.sample config.rb
```

CoreOSはホストマシンとのディレクトリ同期にrsyncを使うっぽい(?)のでWindowsにrsyncをインストール。

- Cygwin使ってる場合はパッケージインストーラを起動して、rsyncを選べばOK。
- MSYS/MinGW使ってるの場合はmingw-getでパッケージインストールできる。
- Git Bashオンリーの場合は実行ファイルやDLLを落としてきて直接配置する。
- Git BashオンリーだがMinGWインストールしてMinGW/binにパスを通す。

私はめんどくさいので４つ目の方法を選択（[参考](http://ktrysmt.github.io/blog/use-mingw-with-git-bash-on-windows/)）。
汚したくない場合（３つ目）は、[ココ](http://hail2u.net/blog/software/install-rsync-to-git-for-windows.html)を参考にすればいい。
最終的にrsyncのバージョン番号が確認できればOK。

```
$ rsync --version
rsync  version 3.0.8  protocol version 30
```

## etcd

外部のVPSサービスやおそらくプロダクションではディスカバリに<https://discovery.etcd.io/>を使うといいようだ。無料で使えるらしい。

だが、今回はローカル環境でやっているので外からは名前解決しない。

なので、<https://discovery.etcd.io/>の代用としてWindows内にetcdを入れてetcdプロセスを立ち上げる。デーモン起動時にオプションでリクエスト流し先のIPを決める必要があり、それにVirtualboxのゲートウェイIPを使う。

ゲートウェイIPはVirtualboxを使う場合はおそらくナンバリングが無い一番上に来るイーサネットアダプタ。

たぶんVirtualboxのゲストマシンから到達できるようにする必要があるからと思う。

いくつかのVMを上げている状態のWindowsで`ipconfig`すると

```
イーサネット アダプター VirtualBox Host-Only Network
　：
（略）
　：
イーサネット アダプター VirtualBox Host-Only Network #2
　：
（略）
　：
イーサネット アダプター VirtualBox Host-Only Network #3
　：
（略）
　：
```

と出るが、これの一番上のアダプタのIP。

### 実行バイナリの入手

<https://github.com/coreos/etcd/releases> から実行バイナリをとってくる。

`OS X`/`Linux`/`Windows`それぞれのバイナリと使い方が書いてあるのでその通りに行う。
Windowsの場合は`etcd-バージョン番号-windows-amd64.zip`のリンクがあるのでそこからzipを入手し解凍。

今回は`etcd-v2.0.9-windows-amd64.zip`を使う。

### 起動

解凍したフォルダに入って以下を実行するとetcdデーモンが立ち上がる。

```
$ cd etcd-v2.0.9-linux-amd64
$ ./etcd -addr <ゲートウェイIP>:4001 -v
```

別のターミナルウィンドウを立ち上げて、動作確認を行う。
立ち上がったデーモンに対して、鍵の登録と登録済み鍵の確認を行い正常に動作していることを確認しておく。
鍵には適当な任意の文字列をいれておく。

```
$ curl http://<ゲートウェイIP>:4001/version
etcd 2.0.9
$ ./etcdctl set mykey "<任意の文字列>"
$ ./etcdctl get mykey
<任意の文字列>
```

ようやく準備が整った。

次回は `vagrant up` でCoreOSたちをまとめて起動する。道のりは長い。


## Reference

- <https://github.com/coreos/coreos-vagrant>
- <https://golang.org/dl/>
- <http://tsubalog.hatenablog.com/entry/how-to-solve-terminal-capability>
- <http://knowledge.sakura.ad.jp/tech/2519/>
- <http://qiita.com/snumano/items/381732bca47918f949df>
- <http://qiita.com/voluntas/items/fb4174b4327c539e1fe7>
- <http://qiita.com/voluntas/items/fc5b992fc3a579029566>



