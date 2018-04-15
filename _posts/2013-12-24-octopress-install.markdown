---
layout: post
title: "[CentOS] OctopressのインストールとGithubPagesでの運用を試す"
date: 2013-12-24 08:47:56 +0000
comments: true
categories: Ruby

---

ためしにやってみた。RubyやGem、Githubに慣れるのも兼ねて。

CentOS 6 です。

## Githubアカウントを作成
作ります。

## Githubアカウント名.github.ioというリポジトリ名で公開リポジトリを作成
作ります。

## rvmとRuby、ライブラリをインストール

```bash
cd /tmp
wget http://ftp.jaist.ac.jp/pub/Linux/Fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
yum install -y gcc-c++ patch readline readline-devel
yum install -y zlib zlib-devel libffi-devel
yum install -y openssl-devel make bzip2 autoconf automake libtool bison
yum install -y gdbm-devel tcl-devel tk-devel
yum install -y libxslt-devel libxml2-devel
yum install -y --enablerepo=epel libyaml-devel
curl -L https://get.rvm.io | bash -s stable
rvm install 2.0.0
ruby -v
yum clean all
```

## Octopressをクローン、bundleとrakeをインストール

```bash
git clone https://github.com/imathis/octopress.git KotaroYoshimatsu.github.io

cd KotaroYoshimatsu.github.io
gem install bundler
bundle install
bundle exec rake install

# ↓URLの入力を求められるのでgithub.ioのURLを入力。
# 例：https://KotaroYoshimatsu@github.com/KotaroYoshimatsu/KotaroYoshimatsu.github.io
bundle exec rake setup_github_pages
```

## 記事作成
post titleは例ですが、この文字列がpermalinkの一部になる、日本語は避けたほうが無難。
記事のタイトルは後から変更可能みたい。
permalinkを後から変えたいときは、ファイル名のYYYY-MM-DD-以降を変えればOK。
permalinkのフォーマットを変えたいときは_config.ymlを編集する。

```bash
rake new_post['post title']

vim source/_post/YYYY-MM-DD-title.markdown # 記事編集

rake preview #localhost:4000でプレビュー可能になる。

rake generate #htmlソースの生成、結構時間がかかる

rake deploy #githubへのデプロイなのでgit pushが実行される

rake gen_deploy #記事のデプロイとジェネレータを同時に実行
```


