---
layout: post
title: "rvmをやめてrbenvでRuby"
date: 2014-07-22 15:09:08 +0900
comments: true
categories: Ruby
published: true
---

## 依存パッケージいれる

```
$ sudo yum -y install git gcc make openssl-devel
```

## インストール

```
$ cd ~
$ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
$ git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
$ source ~/.bash_profile
$ rbenv --version
rbenv 0.4.0-98-g13a474c
```

## Rubyをインストール
まず使いたいバージョンを選ぶ

```
$ rbenv install --list
```

決まったらインストールを実行

```
$ rbenv install 2.1.2
```

## 使用するrubyを設定

システム全体に反映する場合

```
$ rbenv global 2.1.1
$ ruby -v
```

特定のディレクトリ配下で使用する場合

```
$ cd /path/to/dir/
$ rbenv local 2.1.1
$ ruby -v
```
