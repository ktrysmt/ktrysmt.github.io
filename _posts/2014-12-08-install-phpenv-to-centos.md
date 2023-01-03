---
layout: post
title: "CentOS6にphpenvをインストール"
date: 2014-12-08 15:09:08 +0900
comments: true
categories: PHP
published: true
---

昔 phpenv + phpbuild でのセットアップで何かにコケて挫折していました。

PHPのバージョンアップやスイッチイングは非常に煩わしい＋いやな思い出がいっぱいなので、phpenvでPHPの闇とおさらばします。

CentOS6 64bit です。

## Install phpenv

しれっとcurlやgit使ってますが使いますので入れてください。

```
curl https://raw.githubusercontent.com/CHH/phpenv/master/bin/phpenv-install.sh | bash
git clone git://github.com/CHH/php-build.git ~/.phpenv/plugins/php-build
echo 'export PATH="$HOME/.phpenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(phpenv init -)"' >> ~/.bashrc
exec $SHELL -l
```

## Install dependent libraries

いくつか依存がないと警告が出たので入れます。

```
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
yum install -y libxml2 libxml2-devel libjpeg-turbo-devel libpng-devel libpng libmcrypt-devel libtidy-devel libxslt-devel libmcrypt-devel readline-devel openssl-devel curl-devel --enablerepo=epel
```

## Install PHP via phpenv

```
phpenv install -l # バージョン確認
phpenv install 5.5.19 # インストール実行
```

## apply PHP version

```
phpenv local 5.5.19
```

## Reference

- <http://qiita.com/uchiko/items/5f1843d3d848de619fdf>

