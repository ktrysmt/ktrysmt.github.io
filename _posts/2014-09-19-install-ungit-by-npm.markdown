---
layout: post
title: "npmでungitをインストールしてデーモン化"
date: 2014-05-23 15:09:08 +0900
comments: true
categories: Git
published: true
---

## 追記（2014/08/31）nodeならforeverを使ってデーモン化するほうがいいらしい
参考：<http://ktrysmt.github.io/blog/forever-ungit/>

## 追記（2014/08/31）以下、古い情報。

## nvmをいれて、npmを使ってインストール

```
git clone https://github.com/creationix/nvm.git ~/.nvm
source ~/.nvm/nvm.sh
nvm install 0.10
nvm use 0.10
npm install -g ungit
echo "source ~/.nvm/nvm.sh" >> $HOME/.bash_profile
echo "nvm use 0.10" >> $HOME/.bash_profile
```

## パス確認

```
$ which node
$ which ungit
```

## デーモン化

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

## 参考
[サービス起動用スクリプトを作ってみる - いますぐ実践! Linuxシステム管理 / Vol.030](http://www.usupi.org/sysad/030.html)
