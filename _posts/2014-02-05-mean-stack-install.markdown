---
layout: post
title: "MEAN Stackインストール手順まとめ"
date: 2014-02-05 15:09:08 +0900
comments: true
categories: Node
published: true
---

やろうやろうと思って出来なかったmean、
作ってみたいアプリもあるので触ってみることに。

昨日の[NUCON](http://nucon.nulab.co.jp/)によると、typetalkもAngularを使っているとのこと。

## nvmとnodeのインストール

```
# useradd project
# su - project
$ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
$ source $HOME/.bash_profile
$ nvm install 0.10
$ nvm use 0.10
```

## MongoDBのインストール

```
# curl -O http://downloads.mongodb.org/linux/mongodb-linux-x86_64-2.4.9.tgz
# tar -zxvf mongodb-linux-x86_64-2.4.9.tgz
# cd mongodb-linux-x86_64-2.4.9/bin/
# cp -n * /usr/bin/
# useradd mongodb
# mkdir -p /data/mongodb
# chown mongodb /data/mongodb
```

## MongoDBの起動

```
# su - mongodb
$ mongod --dbpath /data/mongodb/ --port 27017
```

デーモン化はあとでやる。

## bowerインストール

```
$ npm install -g bower
```

## Gruntインストール

```
$ npm install -g grunt-cli
```

## mean.ioの出番

```
$ git clone https://github.com/linnovate/mean.git
$ cd mean
$ npm install
$ bower install
$ npm start
```

次はnodeを使ったチャットアプリを実装して、このサンプルに組み込む。

今回参考にしたサイト：

- [creationix/nvm](https://github.com/creationix/nvm)
- [Getting started - Grunt: The JavaScript Task Runner](http://gruntjs.com/getting-started)
- [Install MongoDB on Linux Systems — MongoDB Manual 2.4.9](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-linux/)
- [linnovate/mean](https://github.com/linnovate/mean)

そのあとはAngularのtutorial的なものをパスして、ノウハウにもざっと目を通す。

- [AngularJS Startup Advent Calendar 2013 - Qiita [キータ]](http://qiita.com/advent-calendar/2013/angularjs-startup)

