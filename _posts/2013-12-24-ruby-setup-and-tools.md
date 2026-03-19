---
layout: post
title: "Ruby環境構築まとめ"
date: 2013-12-24 08:47:56 +0000
comments: true
categories: Ruby
published: true
description: "Ruby環境構築の総合ガイド。rvm/rbenvによるインストール、gemsetの使い方、Octopress導入、Capybaraテスト環境、便利ツール紹介"
redirect_from:
  - /blog/octopress-install/
  - /blog/use-gemset/
  - /blog/install-ruby-by-rbenv-to-centos/
  - /blog/rubykaigi-synvert-transpec-rubocop/
  - /blog/setup-capybara-with-phantomjs/
tags:
  - ruby
  - git
  - centos
  - linux
  - testing
---

Ruby環境構築に関する知見をまとめた記事です。rvmやrbenvによるインストール、gemsetの使い方、Octopressの導入、テスト環境の構築、便利ツールの紹介などを網羅しています。

## CentOSでOctopressをインストールしGitHub Pagesで運用する

ためしにやってみた。RubyやGem、Githubに慣れるのも兼ねて。

CentOS 6 です。

### Githubアカウントを作成
作ります。

### Githubアカウント名.github.ioというリポジトリ名で公開リポジトリを作成
作ります。

### rvmとRuby、ライブラリをインストール

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

### Octopressをクローン、bundleとrakeをインストール

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

### 記事作成
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

## rvmのgemsetでrubyとrailsのバージョンを管理する

### rvmでrubyとrailsをセット

まずはrvmを更新しておく。

```
[root@localhost ~]# rvm get stable
```

今回必要なバージョンはそれぞれ以下。

``` bash
ruby 1.8.7 (2009-12-24 patchlevel 248) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 2010.01
Rails 2.3.5
```

### 1.8.7をインストールしgemset（受け皿）を作成

``` bash
[root@localhost ~]# rvm install 1.8.7
Warning! PATH is not properly set up, '/usr/local/rvm/gems/ruby-1.9.3-p374/bin' is not at first place,
         usually this is caused by shell initialization files - check them for 'PATH=...' entries,
         it might also help to re-add RVM to your dotfiles: 'rvm get stable --auto-dotfiles',
         to fix temporarily in this shell session run: 'rvm use ruby-1.9.3-p374'.
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/6/x86_64/ruby-1.8.7-p374.
It is not possible to build movable binaries for rubies 1.8-1.9.2, but you can do it for your system only.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Updating system.
Installing required packages: patch, patch, readline-devel, libyaml-devel, libffi-devel, libtool, bison/
........
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-1.8.7-p374, this may take a while depending on your cpu(s)...
ruby-1.8.7-p374 - #downloading ruby-1.8.7-p374, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 4150k  100 4150k    0     0  10.4M      0 --:--:-- --:--:-- --:--:-- 15.4M
ruby-1.8.7-p374 - #extracting ruby-1.8.7-p374 to /usr/local/rvm/src/ruby-1.8.7-p374.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/1.8.7/stdout-rouge-fix.patch.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/1.8.7/no_sslv2.diff.
ruby-1.8.7-p374 - #applying patch /usr/local/rvm/patches/ruby/ssl_no_ec2m.patch.
ruby-1.8.7-p374 - #configuring...............................
ruby-1.8.7-p374 - #post-configuration.
ruby-1.8.7-p374 - #compiling..........................................
ruby-1.8.7-p374 - #installing.
ruby-1.8.7-p374 - #making binaries executable.
ruby-1.8.7-p374 - #downloading rubygems-2.0.14
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  329k  100  329k    0     0  1495k      0 --:--:-- --:--:-- --:--:-- 7161k
ruby-1.8.7-p374 - #extracting rubygems-2.0.14.
ruby-1.8.7-p374 - #removing old rubygems.
ruby-1.8.7-p374 - #installing rubygems-2.0.14.........................
Error running 'env GEM_PATH=/usr/local/rvm/gems/ruby-1.9.3-p374:/usr/local/rvm/gems/ruby-1.9.3-p374@global:/usr/local/rvm/gems/ruby-1.9.3-p374:/usr/local/rvm/gems/ruby-1.9.3-p374@global GEM_HOME=/usr/local/rvm/gems/ruby-1.9.3-p374 /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/ruby -d /usr/local/rvm/src/rubygems-2.0.14/setup.rb',
showing last 15 lines of /usr/local/rvm/log/1389229013_ruby-1.8.7-p374/rubygems.install.log
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/Class3PublicPrimaryCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/Class3PublicPrimaryCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/EntrustnetSecureServerCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/EntrustnetSecureServerCertificationAuthority.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/DigiCertHighAssuranceEVRootCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/DigiCertHighAssuranceEVRootCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/GeoTrustGlobalCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/site_ruby/1.8/rubygems/ssl_certs/GeoTrustGlobalCA.pem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1427 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/gem
Exception `Errno::ENOENT' at /usr/local/rvm/rubies/ruby-1.8.7-p374/lib/ruby/1.8/fileutils.rb:1304 - No such file or directory - /usr/local/rvm/rubies/ruby-1.8.7-p374/bin/gem
Exception `Gem::InstallError' at ./lib/rubygems/uninstaller.rb:101 - gem "gemcutter" is not installed
/usr/local/rvm/gems/ruby-1.9.3-p374/gems/json-1.7.7/lib/json/ext/parser.so: [BUG] Segmentation fault
ruby 1.8.7 (2013-06-27 patchlevel 374) [x86_64-linux]

RubyGems 2.0.14 installed
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374@global
ruby-1.8.7-p374 - #importing gemset /usr/local/rvm/gemsets/global.gems......
ruby-1.8.7-p374 - #generating global wrappers.
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374
ruby-1.8.7-p374 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-1.8.7-p374 - #generating default wrappers.
ruby-1.8.7-p374 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-1.8.7-p374 - #complete
WARNING: Please be aware that you just installed a ruby that is no longer maintained, for a list of maintained rubies visit:

    http://bugs.ruby-lang.org/projects/ruby/wiki/ReleaseEngineering

Please consider upgrading to ruby-2.1.0 which will have all of the latest security patches.
Ruby was built without documentation, to build it run: rvm docs generate-ri

[root@localhost ~]# rvm gemset create rails235
ruby-1.8.7-p374 - #gemset created /usr/local/rvm/gems/ruby-1.8.7-p374@rails235
ruby-1.8.7-p374 - #generating rails235 wrappers.
[root@localhost ~]# rvm 1.8.7@rails235
```

### 作ったgemsetでrailsをインストール。

``` bash
# gem install rails --version "=2.3.5"
```

同一バージョンのrubyで複数のgemsetを作成して管理することもできるらしい。
参考にしたのはこちら。
http://serihiro.hatenablog.com/entry/20130421/1366552174

マシンを分けたほうがいいと思うけど、必要な場合もあるだろうということで。

## rvmをやめてrbenvでRuby

### 依存パッケージいれる

```
$ sudo yum -y install git gcc make openssl-devel
```

### インストール

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

### Rubyをインストール
まず使いたいバージョンを選ぶ

```
$ rbenv install --list
```

決まったらインストールを実行

```
$ rbenv install 2.1.2
```

### 使用するrubyを設定

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

## Rubykaigiで話題のツール紹介

あとでインストールして使ってみる。

### synvert

+ [xinminlabs/synvert](https://github.com/xinminlabs/synvert)

### transpec

+ [yujinakayama/transpec](https://github.com/yujinakayama/transpec)

### rubocop

+ [bbatsov/rubocop](https://github.com/bbatsov/rubocop)
+ [Ruby - Rubocopを使ってコーディングルールへの準拠チェックを自動化 - Qiita](http://qiita.com/yaotti/items/4f69a145a22f9c8f8333)
+ [bbatsov/rubocop](https://github.com/bbatsov/rubocop#editor-integration)
+ [ngmy/vim-rubocop](https://github.com/ngmy/vim-rubocop)

## phantomJSとcapybaraを使ったテスト環境を構築する

環境はCentoOS6 です。

### install

#### PhantomJS

過去の記事を参考に。

- <http://ktrysmt.github.io/blog/how-to-install-phantomjs-casperjs/>

キャプチャを取らない場合はFontのインストールはしなくてよい。

#### Ruby

rbenvを使います。これも過去の記事を参考にインストール。

+ <http://ktrysmt.github.io/blog/install-ruby-by-rbenv-to-centos/>

Rubyのバージョンはなんでもいいけど比較的新しくて安定版であればいいと思う。

今回は`2.1.3`を使ってみる（2014年9月時点）。

### setup

#### Gemfile

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

#### 依存ライブラリのインストール

`bundle install`したら依存ライブラリがあるといわれたので。

``` sh
sudo yum -y install libxslt libxml2
```
