---
layout: post
title: "CentOS6にGerritをインストール"
date: 2015-02-27 15:09:08 +0900
comments: true
categories: Git
published: true
---

## Install Java

### 下記を参考にJavaを設置

- <http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/>

```
cd /opt/
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u71-b14/jdk-7u71-linux-x64.tar.gz"
tar xzf jdk-7u71-linux-x64.tar.gz
cd /opt/jdk1.7.0_71/
alternatives --install /usr/bin/java java /opt/jdk1.7.0_71/bin/java 2
alternatives --config java
```

### 対話形式になるので`1`を入力しEnter

```
1 プログラムがあり 'java' を提供します。

  選択       コマンド
-----------------------------------------------
*+ 1           /opt/jdk1.7.0_71/bin/java

Enter を押して現在の選択 [+] を保持するか、選択番号を入力します:1 <ENTER>
```

### インストールを確認

```
java -version
```

### 以下のように表示されればOK

```
java version "1.7.0_71"
Java(TM) SE Runtime Environment (build 1.7.0_71-b14)
Java HotSpot(TM) 64-Bit Server VM (build 24.71-b01, mixed mode)
```

### 環境変数設定を設定。

bashrcやbash_profileに入れておく。

```
vim ~/.bashrc
```

```
export JAVA_HOME=/opt/jdk1.7.0_71
export JRE_HOME=/opt/jdk1.7.0_71/jre
export PATH=$PATH:/opt/jdk1.7.0_71/bin:/opt/jdk1.7.0_71/jre/bin
```


## Gerritのインストール

### 以下を参考にセットアップを進める

- <https://gerrit-review.googlesource.com/Documentation/install-quick.html>

### ソースを取得

以下から最新のwarを取得する

- <http://code.google.com/p/gerrit/downloads/detail?name=gerrit-2.2.2.war>

### Setup

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

### Configure SSH

```
ssh-keygen
```

# 参考

- <http://tecadmin.net/steps-to-install-java-on-centos-5-6-or-rhel-5-6/>
- <https://gerrit-review.googlesource.com/Documentation/install-quick.html>
