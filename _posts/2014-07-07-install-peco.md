---
layout: post
title: "[bash]CentOSにpecoをインストール"
date: 2014-05-24 15:09:08 +0900
comments: true
categories: Golang
published: true
---

## 追記（2014-09-06）pecoのバイナリを設置してパスを通すだけでいい
pecoのバイナリがあったようで、goは不要だそう。

## install peco

リリースノートから欲しいバージョンのpecoを選択してDLすればよい。

+ <https://github.com/peco/peco/releases>

CentOSで使いたいのでlinux版を選択。

```
$ wget https://github.com/peco/peco/releases/download/v0.2.9/peco_linux_amd64.tar.gz
$ tar -xzvf peco_linux_amd64.tar.gz
$ sudo mv peco_linux_amd64/peco /usr/local/bin/
$ ls | peco
```

## bashのCtrl+rにpecoを割り当て
[bash のヒストリを peco で便利にする - Qiita](http://qiita.com/comutt/items/f54e755f22508a6c7d78)

Ctrl+rで呼び出し、Ctrl+mで貼り付ける。

```
$ vim $HOME/.bashrc

_replace_by_history() {
    declare l=$(HISTTIMEFORMAT= history | sort -k1,1nr | perl -ne 'BEGIN { my @lines = (); } s/^\s*\d+\s*//; $in=$_; if (!(grep {$in eq $_} @lines)) { push(@lines, $in); print $in; }' | peco --query "$READLINE_LINE")
    READLINE_LINE="$l"
    READLINE_POINT=${#l}
}
bind -x '"\C-r": _replace_by_history'
bind    '"\C-xr": reverse-search-history'
```

```
$ source .bashrc
```

## 参考
+ <http://qiita.com/fortkle/items/52c1077a7963cb01c596>
+ <http://qiita.com/puttyo_bubu/items/b89ffd75e2012a0035ac>
+ <http://qiita.com/comutt/items/f54e755f22508a6c7d78>
+ <https://github.com/peco/peco/releases>
+ <http://qiita.com/lestrrat/items/de8565fe32864f76ac19>





## 追記（2014-09-06）以下、古い情報。

## install go
ソース解凍しパスを通せば良い。

```
$ wget https://storage.googleapis.com/golang/go1.2.2.linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
$ echo "export GOPATH=/usr/local/go" >> $HOME/.bashrc
$ echo "export PATH=$PATH:$GOPATH/bin" >> $HOME/.bashrc
$ go version
go version go1.2.2 linux/amd64
```

反映。

```
$ source .bashrc
```
