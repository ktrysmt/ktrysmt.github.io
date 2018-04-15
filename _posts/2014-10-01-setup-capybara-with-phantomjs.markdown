---
layout: post
title: "phantomJSとcapybaraを使ったテスト環境を構築する"
date: 2014-09-27 15:09:08 +0900
comments: true
categories: Ruby
published: true
---

環境はCentoOS6 です。

## install

### PhantomJS

過去の記事を参考に。

- <http://kotaroyoshimatsu.github.io/blog/how-to-install-phantomjs-casperjs/>

キャプチャを取らない場合はFontのインストールはしなくてよい。

### Ruby

rbenvを使います。これも過去の記事を参考にインストール。

+ <http://kotaroyoshimatsu.github.io/blog/install-ruby-by-rbenv-to-centos/>

Rubyのバージョンはなんでもいいけど比較的新しくて安定版であればいいと思う。

今回は`2.1.3`を使ってみる（2014年9月時点）。

## setup

### Gemfile

Gemfileの設置とインストールをします。

``` sh
$ vim Gemfile
```

``` ruby
source 'https://rubygems.org'
gem 'capybara'
gem 'rspec'
gem 'launchy'
gem 'poltergeist'
```

``` sh
gem install bundler
source ~/.bash_profile
bundle install
```

### 依存ライブラリのインストール

`bundle install`したら依存ライブラリがあるといわれたので。

``` sh
sudo yum -y install libxslt libxml2
```
