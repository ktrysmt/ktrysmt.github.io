---
layout: post
title: "Windows7でdockerを使いプロビジョニングテストするまで"
date: 2015-03-01 15:09:08 +0900
comments: true
categories: Vagrant
published: true
---

ansibleやServerspecのテストをしたいので、いくらでも気軽に使い捨てられる仮想マシンがほしい。

オフラインで作業することも考慮すると、できればまずは手元のマシンだけで完結できるようにしておきたい。

一方でそろそろDocker熱も落ち着いてきてノウハウがだいたいまとまってきた頃だと思われる。

よって、最短で使えるようにひと通りの手順をここらでまとめてみる。

## Tools

- VirtualBox (>=4.3.10)
- Vagrant (>=1.6)
- Windows7 64bit
- GitBash
- Cmder
- boot2docker

## How to Setup

前述のToolsをすべてインストールしたらWindowsを再起動

### 1. Setup boot2docker

boot2dockerを起動

```
$ boot2docker.exe version
Boot2Docker-cli version: v1.5.0
Git commit: ccd9032
```

パスが通っていればOK。

このboot2dockerコマンドを使いdockerホストマシンを起動する。

boot2dockerコマンドはboot2docker-vmという仮想マシンをVirtualBox上に作成する。

```
$ boot2docker.exe init # VirtualBox内にboot2docker-vmというマシンを作っている
$ boot2docker.exe up # 作ったマシンを起動
$ boot2docker.exe ssh # 作ったマシンにssh接続
docker@boot2docker:~$ # 接続成功
```

### 2. make Dockerfile

<https://gist.github.com/kazuph/8064771>を参考にしつつDockerfileを作成。

centos:latestはCentOS7なので普段使うCentOS6をFROMに指定してる。

```
FROM centos:centos6

RUN yum install -y passwd
RUN yum install -y openssh
RUN yum install -y openssh-server
RUN yum install -y openssh-clients
RUN yum install -y sudo

RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config
RUN service sshd start
RUN service sshd stop

RUN echo 'root:docker' |chpasswd

RUN useradd docker
RUN echo 'docker:docker' |chpasswd
RUN echo "docker    ALL=(ALL)       ALL" >> /etc/sudoers.d/docker

CMD /usr/sbin/sshd -D
```

Rootでパス無しログインさえできればいい場合はもっとシンプル

```
FROM centos:centos6

RUN yum -y install openssh-server openssh-clients

RUN echo 'root:' | chpasswd
RUN sed -ri 's/.*PermitRootLogin.+/PermitRootLogin yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
RUN sed -ri 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config
RUN sed -ri 's/#PermitEmptyPasswords no/PermitEmptyPasswords yes/g' /etc/ssh/sshd_config

RUN service sshd start

CMD /usr/sbin/sshd -D
```

### 3. Dockerfileの共有

最新のboot2dockerであればデフォルトでWindowsの`C:\Users`以下がboot2docker-vm内の`/c/Users`と共有される設定になっている。ここにDockerfileを作りboot2docker-vm内で`docker build`するときに使用する。GitやSCPなんかでvm内から外へ転送・共有するのが面倒なので。

作ったDockerfileをboot2docker-vmで参照できるように共有フォルダ配下に保存する。

今回は適当に`C:\Users\username\Documents`以下にdockerというフォルダを作ってそこに置いておく。

あとはboot2docker-vmにログインしてdocker build

```
docker@boot2docker:~$ docker build -t centos:sshd /c/Users/username/Documents/docker/
```

### 4. docker run

dockerをポート番号指定して起動する。

```
docker@boot2docker:~$ docker run -d -p 10022:22 centos:sshd
```

### 5. コンテナにsshする

まずはホストマシンからsshできることを確認。

```
$ ssh root@`boot2docker.exe ip` -p 10022
```

次に別のVirtualBoxからもsshできるようにしたいので以下の手順で設定。

+ boot2dockerを止める
+ VirtualBoxのGUIを開く
+ 設定 → ネットワークの画面を開く
+ アダプター 3を有効にする
+ 割り当てをブリッジアダプターにする
+ OKボタンを押下して設定を保存
+ boot2dockerを起動
+ boot2docker ssh
+ boot2docker-vm内でifconfigして3つめのNIC(eth2)のipアドレスを把握
+ 把握したIPに他のホストからアクセス

### 6. 作業内容の保存と復元

docker commit を使えば作業中の状態をimageとして保存できる。保存時、`centos:sshd-sample` とすれば`リポジトリ名:タグ名`として新しいimageを作成できる。

```
docker commit <コンテナID> centos:sshd-sample
```

作成したimageを指定して`docker run` すれば、またその状態を復元できる。

```
docker run -d -p 10023:22 centos:sshd-sample
```

検証作業や構築作業が途中な場合はいったんコミットしておいて、構築手順が固まったらDockerfileに正式なものとして書き込むという運用にすればいい。

### 総評

Vagrantを使うより起動が早かったりするのと、commit>run>Dockerfileのサイクルが身につくと、日々プロビジョニングを意識して日常の作業を行う癖がついたりして、事故があったときの復旧速度が格段に上がる。ひとつのVMにいろんなアプリケーションを詰め込んでいる状態から離脱もしやすくなって、精神的に健全。アプリケーションごとにコンテナをわけるというMicroServices的発想もにも自然となりやすく、ともすれば複雑になりがちなシステム管理に見通しが立てやすくなる。

ただDocker pullが不満だ。Quay.ioもいいが有料だ。そこだけspofになってしまうというかベンダーロックインされてしまう点がどうも気に食わない。今はプロダクションで使うところまで行っていないので特に問題は感じないが、いずれぶち当たる壁。

いっそgoに舵をきってバイナリを扱うようにしてしまうとか、とにかくimagesは小さくまとめつつ取り回ししやすいようにしていきたい。

- [CoreOS 上に Private Docker Registry を立てる - Qiita](http://qiita.com/spesnova/items/658a47e40eaea5b5a5f4)

## 参考

- <http://j-caw.co.jp/blog/?p=1561>
- <http://knowledge.sakura.ad.jp/tech/1811/>
- <http://qiita.com/nyamage/items/fad845bc5e4ce3cf33eb>
