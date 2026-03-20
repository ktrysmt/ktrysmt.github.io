---
layout: post
title: "PHP環境構築まとめ (CentOS)"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: Programming
published: true
description: "CentOSでのPHP環境構築をまとめた記事。yumやphpenvでのインストール、バージョン管理、php-cs-fixerによるコード整形まで網羅"
redirect_from:
  - /blog/install-php5-centos/
  - /blog/install-phpenv-to-centos/
  - /blog/use-php-cs-fixer-with-git-hook-pre-commit/
  - /blog/install-php5.2-into-centos5/
tags:
  - php
  - centos
  - linux
  - git
  - cli-tools
---

CentOSにおけるPHP環境構築に関する知見をまとめた記事です。yumによるバージョンアップ、phpenvの導入、php-cs-fixerの活用、レガシー環境への対応などを扱います。

## CentOS6でPHP 5.3から5.5にバージョンアップする

CentOS6 64bitです。

### uninstall PHP 5.3

```
yum list installed php* # アンインストールしたいのをメモっとく
yum remove php*
yum list installed php* # アンインストールされたか確認
```

### add 3rd repositories

install

```
cd
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
rpm -Uvh remi-release-6.rpm
rpm -Uvh rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
```

switch

```
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/rpmforge.repo
sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/remi.repo
```

### check php55

```
yum list php* --enablerepo=remi-php55
```

### install php55

remove前にメモっておいたのを入れるようにするといいが、無事故は保証できない。

```
yum install --enablerepo=remi-php55 php php-devel ... #いれたいものをいれる
```

自分がよく使うのは以下。

```
yum install --enablerepo=epel,remi-php55 php-devel php-fpm php-mysql php-mbstring
```

## CentOS6にphpenvをインストール

昔 phpenv + phpbuild でのセットアップで何かにコケて挫折していました。

PHPのバージョンアップやスイッチイングは非常に煩わしい＋いやな思い出がいっぱいなので、phpenvでPHPの闇とおさらばします。

CentOS6 64bit です。

### Install phpenv

しれっとcurlやgit使ってますが使いますので入れてください。

```
curl https://raw.githubusercontent.com/CHH/phpenv/master/bin/phpenv-install.sh | bash
git clone git://github.com/CHH/php-build.git ~/.phpenv/plugins/php-build
echo 'export PATH="$HOME/.phpenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(phpenv init -)"' >> ~/.bashrc
exec $SHELL -l
```

### Install dependent libraries

いくつか依存がないと警告が出たので入れます。

```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
yum install -y libxml2 libxml2-devel libjpeg-turbo-devel libpng-devel libpng libmcrypt-devel libtidy-devel libxslt-devel libmcrypt-devel readline-devel openssl-devel curl-devel --enablerepo=epel
```

### Install PHP via phpenv

```
phpenv install -l # バージョン確認
phpenv install 5.5.19 # インストール実行
```

### apply PHP version

```
phpenv local 5.5.19
```

### Reference

- <http://qiita.com/uchiko/items/5f1843d3d848de619fdf>

## git-hookのpre-commitでphp-cs-fixerを叩く

php-cs-fixerはphp5.3以上を要求するので注意。

インストールはcomposer経由でもなんでも。

とりあえずリポジトリのROOTにpharおいてしまうという大雑把なやり口なのであまり真似しないように。

### Install

```
wget http://get.sensiolabs.org/php-cs-fixer.phar
mv php-cs-fixer{.phar,}
chmod +x php-cs-fixer
```

### Edit .git/hooks/pre-commit

```
touch .git/hooks/pre-commit
chmod +x !$
```

で、`.git/hooks/pre-commit`に以下を追加

```
#!/bin/bash

# ディレクトリ定義
ROOT_DIR=$(git rev-parse --show-toplevel)
LIST=$(git status | grep -e '\(modified:\|new file:\)'| grep '\.php' | cut -d':' -f2 )

# php-cs
for file in $LIST
do
  RESULT=$($ROOT_DIR/php-cs-fixer --level=psr2 fix $ROOT_DIR$file --fixers=-psr0)
done
```

### 備考

古いFWだと名前空間という概念がないおかげでクラス名が非常に重要だったりする。
つまりクラス名を勝手に書き換えられるのは困るので`-psr0`としている。

## CentOS5にPHP5.2を入れなければならない

遭遇するたびにググってて時間の無駄なのでコピペ用にまとめ。

```
echo '[utter] \
name=Jason'"'"'s Utter Ramblings Repo \
baseurl=http://www.jasonlitka.com/media/EL$releasever/$basearch/ \
enabled=0 \
gpgcheck=1 \
gpgkey=http://www.jasonlitka.com/media/RPM-GPG-KEY-jlitka' > /etc/yum.repos.d/utterramblings.repo
yum --enablerepo=utter -y install php php-devel php-mbstring php-mysql php-xml php-gd php-mcrypt mysql mysql-server
```

`yum remove php* mysql*`してから行う場合はmysql-libs消すときの巻き込みに注意。
postfixとcrontabsもremoveされてしまう。
