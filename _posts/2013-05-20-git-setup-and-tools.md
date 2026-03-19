---
layout: post
title: "Git環境構築・ツールまとめ"
date: 2013-05-20 12:00:00 +0900
comments: true
categories: Git
published: true
description: "CentOSへのGitインストール、diff-highlight、tig、ghq、ungit、Gerritなど、Git関連ツールの導入手順をまとめて紹介"
tags:
  - git
  - centos
  - linux
  - cli-tools
  - tig
  - ghq
  - golang
  - node-js
  - javascript
redirect_from:
  - /blog/install-Git-1.8-into-CentOS/
  - /blog/diff-highlight-visualize-git/
  - /blog/tig-install-centos/
  - /blog/install-ghq/
  - /blog/forever-ungit/
  - /blog/install-ungit-by-npm/
  - /blog/install-gerrit-to-centos6/
---

## CentOSにGit 1.8をインストール

yumでのインストールだと1.7系までみたいなので、1.8は手でインストールする必要がある。

個人的に使いたいものがどれも1.8委譲を要求するので、仕方ないが入れる。

```
# wget https://git-core.googlecode.com/files/git-1.8.5.2.tar.gz
# tar zxvf git-1.8.5.2.tar.gz
# cd git-1.8.5.2
# ./configure
# make
# make install
```

インストールされたことを確認。

```
$ which git
/usr/local/bin/git
$ git --version
git version 1.8.2.1
```

### 参考
- [CentOS 6.4にgitの最新版をソースから入れる - /dev/null](http://gitpub.hatenablog.com/entry/2013/07/04/001010)

## gitのdiffを見やすくするdiff-highlight

[Git の diff を美しく表示するために必要なたった 1 つの設定 #git - 詩と創作・思索のひろば (Poetry, Writing and Contemplation)](http://motemen.hatenablog.com/entry/2013/11/26/Git_%E3%81%AE_diff_%E3%82%92%E7%BE%8E%E3%81%97%E3%81%8F%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB%E5%BF%85%E8%A6%81%E3%81%AA%E3%81%9F%E3%81%A3%E3%81%9F_1_%E3%81%A4%E3%81%AE%E8%A8%AD)

に触発されたのだが、
いまいちサクッと入れられなかったので、自分の環境で試した時の手順を以下にまとめておく。

Gitのバージョンが古いとおそらくdiff-highlightがGitインストール時に同封されていないので、1.8系へのバージョンアップを試みる。


```
$ vi ~/.gitconfig
```

```
[pager]
        log = diff-highlight | less
        show = diff-highlight | less
        diff = diff-highlight | less
```

diff-highlightを探す。

```
$ locate diff-highlight
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight/README
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight/diff-highlight # 見つけた
```

実行できるようにパスを通す

```
$ ln -s /usr/share/doc/git-1.8.2.1/contrib/diff-highlight/diff-highlight /usr/bin/
```

`git diff`してもカラーリングされない、白黒のままだ、っていう場合は`git config`でのカラーリングの設定ができていない。

```
$ git config --global color.ui true
```

## CentOSにtigをインストール

### ソースからインストール

```
cd /tmp
git clone git://github.com/jonas/tig.git
cd tig/
make prefix=/usr
make install prefix=/usr
```

### パッケージからインストール

CentOS6ならパッケージを使うほうがよい。

rpmforgeを使う。

```
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
yum list | grep tig | grep rpmforge # 存在確認
yum install -y tig --enablerepo=rpmforge # インストール
tig --version # インストール確認
```

## CentOS 6.5にghqをインストールする

### install go
参考：<http://ktrysmt.github.io/install-peco/>

### install ghq

```
$ yum -y install hg
$ go get github.com/motemen/ghq
$ which ghq
```

## npmでungitをインストールしてデーモン化

### 追記（2014/08/31）nodeならforeverを使ってデーモン化するほうがいいらしい
参考：<http://ktrysmt.github.io/blog/forever-ungit/>

### 追記（2014/08/31）以下、古い情報。

### nvmをいれて、npmを使ってインストール

```
git clone https://github.com/creationix/nvm.git ~/.nvm
source ~/.nvm/nvm.sh
nvm install 0.10
nvm use 0.10
npm install -g ungit
echo "source ~/.nvm/nvm.sh" >> $HOME/.bash_profile
echo "nvm use 0.10" >> $HOME/.bash_profile
```

### パス確認

```
$ which node
$ which ungit
```

### デーモン化

```
$ vi /etc/init.d/ungit
```

```
#!/bin/sh
# chkconfig: 2345 91 91

. /etc/rc.d/init.d/functions

PROG="/root/.nvm/v0.11.13/bin/ungit"
PROGNAME=`basename $PROG`

[ -f $PROG ] || exit 0

case "$1" in
    start)
        echo -n $"Starting $PROGNAME:"
        daemon $PROG
        echo
        ;;
    stop)
        echo -n $"Stopping $PROGNAME:"
        killproc $PROGNAME
        echo
        ;;
    status)
        PROC_STATUS=`ps aux | grep ungit | grep -v grep`
        if [ $PROC_STATUS ] ; then
          echo -n $"$PROGNAME Started..."
        fi
        echo
        ;;
    *)
        echo $"Usage: $PROGNAME {start|stop|status}" >&2
        exit 1
        ;;
esac
exit 0
```

```
$ chkconfig --add ungit
$ chkconfig ungit on
```

### 参考
[サービス起動用スクリプトを作ってみる - いますぐ実践! Linuxシステム管理 / Vol.030](http://www.usupi.org/sysad/030.html)

## foreverでungitをデーモン化

デーモン書いていましたがこちらのほうがスマートな印象。
再起動も自動でしてくれるようです。

nodeとnpmがインストールされているのを前提に。

```
npm -g install forever ungit
echo "forever `which ungit` > /dev/null &" >> ~/.bash_profile
```

### 反映

```
source ~/.bash_profile
```

## CentOS6にGerritをインストール

### Install Java

#### 下記を参考にJavaを設置

- <http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/>

```
cd /opt/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u71-b14/jdk-7u71-linux-x64.tar.gz"
tar xzf jdk-7u71-linux-x64.tar.gz
cd /opt/jdk1.7.0_71/
alternatives --install /usr/bin/java java /opt/jdk1.7.0_71/bin/java 2
alternatives --config java
```

#### 対話形式になるので`1`を入力しEnter

```
1 プログラムがあり 'java' を提供します。

  選択       コマンド
-----------------------------------------------
*+ 1           /opt/jdk1.7.0_71/bin/java

Enter を押して現在の選択 [+] を保持するか、選択番号を入力します:1 <ENTER>
```

#### インストールを確認

```
java -version
```

#### 以下のように表示されればOK

```
java version "1.7.0_71"
Java(TM) SE Runtime Environment (build 1.7.0_71-b14)
Java HotSpot(TM) 64-Bit Server VM (build 24.71-b01, mixed mode)
```

#### 環境変数設定を設定。

bashrcやbash_profileに入れておく。

```
vim ~/.bashrc
```

```
export JAVA_HOME=/opt/jdk1.7.0_71
export JRE_HOME=/opt/jdk1.7.0_71/jre
export PATH=$PATH:/opt/jdk1.7.0_71/bin:/opt/jdk1.7.0_71/jre/bin
```

### Gerritのインストール

#### 以下を参考にセットアップを進める

- <https://gerrit-review.googlesource.com/Documentation/install-quick.html>

#### ソースを取得

以下から最新のwarを取得する

- <http://code.google.com/p/gerrit/downloads/detail?name=gerrit-2.2.2.war>

#### Setup

ディレクトリ名 `~/gerrit_testsite` は適宜変更。

```
# cd ~/
# wget https://gerrit.googlecode.com/files/gerrit-2.2.2.war
# java -jar gerrit-2.2.2.war init --batch -d ~/gerrit_testsite
Generating SSH host key ... rsa(simple)... done
Initialized /home/gerrit2/gerrit_testsite
Executing /home/gerrit2/gerrit_testsite/bin/gerrit.sh start
Starting Gerrit Code Review: OK
# git config -f ~/gerrit_testsite/etc/gerrit.config gerrit.canonicalWebUrl
http://localhost:8080/
# git config -f ~/gerrit_testsite/etc/gerrit.config gerrit.canonicalWebUrl http://YOUR_DOMAIN_NAME:8080/
# git config -f ~/gerrit_testsite/etc/gerrit.config auth.type HTTP
```

ここまでで`http://YOUR_DOMAIN_NAME:8080/`にブラウザでアクセスできるようになる。

#### Configure SSH

```
ssh-keygen
```

### 参考

- <http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/>
- <https://gerrit-review.googlesource.com/Documentation/install-quick.html>
