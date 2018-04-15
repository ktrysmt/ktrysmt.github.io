---
layout: post
title: "PHPもお洒落になったもんだなー"
date: 2015-02-14 15:09:08 +0900
comments: true
categories: PHP
published: true
---



- バージョンアップしたらeregとか非推奨関数を使いまくってて悩ましい
- ライブラリ？使えそうなものは使え、使えなくなったら知らん
- var_dumpで一行debugしてブラウザで確認

（var_dump連発は今も使うけど）昔は大変だったなぁ。

## で、最近は便利になったらしい

- phpbrew ... PHPバージョン管理。nodebrew,rbenvみたいな
- composer ... パッケージ管理。gem,npmに似てる
- PsySH ... インタラクティブシェル。pry的な

## install

CentOS6で、私は普段はzshを使用しています。

```
# phpbrewにはphpが必要
yum install php

# phpbrew
wget http://raw.github.com/c9s/phpbrew/master/phpbrew --no-check-certificate
chmod +x phpbrew
mv phpbrew /usr/local/bin/
source ~/.phpbrew/bashrc
echo "source ~/.phpbrew/bashrc" >> ~/.zshrc

# composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
source ~/.zshrc

# psysh
composer global require psy/psysh
```

## setup phpbrew

コマンドの使い方や詳細は<https://github.com/phpbrew/phpbrew>を参考に。

よく使う構成はPHP5.5,MySQL,php-fpmあたりなのでそれを入れる。

ビルドには数分かかるようです。

```
# libraries
yum -y install openssl-devel bzip2-devel libcurl

# init
phpbrew init

# test
phpbrew -d install --test 5.5.20 +default +mysql +fpm -- --with-openssl

# build
phpbrew -d install 5.5.20 +default +mysql +fpm -- --with-openssl

# add extensions
phpbrew ext install opcache
phpbrew ext install memcache

# use (for temporary)
phpbrew use php-5.5.20

# off (homebrewを不使用にできる)
phpbrew off

# switch (システム標準のPHPをphpbrewに)
phpbrew switch php-5.5.20
```

`--test`オプションでテストもできる。
`-d`オプションでインストール時のオプションが表示されるので、ビルドに失敗した時はオプションを確認しつつどこで止まったのかをビルドログから確認。

## PsySHを使う

composerで入れたので、binをPATHに入れるか直接叩きます。

```
~/.composer/vendor/bin/psysh
```

## configure php, php-fpm

### php.ini, php-fpm.ini

php.iniとphp-fpm.iniの場所を確認。

```
vim ~/.phpbrew/php/php-5.5.20/etc/php.ini
vim ~/.phpbrew/php/php-5.5.20/etc/php-fpm.ini
```

最低限、php.iniではTimezoneを、
php-fpm.iniではuser,group,confのinclude
あたりを設定、有効にしておく。

### setup php-fpm confs

```
mkdir ~/.phpbrew/php/php-5.5.20/etc/fpm.d/
vim ~/.phpbrew/php/php-5.5.20/etc/fpm.d/virtual.conf
```

### service php-fpm restart

php-fpmの起動スクリプトを設置。

```
cp ~/.phpbrew/build/php-5.5.20/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
service php-fpm status
service php-fpm restart
```

## 雑感

phpbrewがphpenvとどう違うのかを詳しく調べられていないが、phpbrewのほうがビルドは早かったのと、php-fpmとかExtensionとかの取り回しがしやすかった、というかgithubのREADMEがわかりやすくて使い勝手が良い。今後バージョン管理するならphpbrewを使っていくと思う。

ただインフラ使い捨てな昨今、別にyumでいつもどおりremi-php55使えば済む話なんだよな。

ではあるんだが、phpに関する依存や細かな問題を一箇所に丸め込められるのは無視できない。+default +mysql +php-fpmとかでフロントエンドとして一括管理できると将来的にも余計なことを考えなくて済むようになる。php-mbstring入れ忘れてたわとか言っていちいちググるのとかほんと時間もったいない。

今はyumをコピペで良くても、いずれPHPがバージョンアップしてきたり新しいソフトウェアが台頭してきた時にはまた手順書を仕上げる作業が発生する。

開発にはますますスピードが求められてくるので、そういうややこしそうな問題はなるべく優れたツールに押し込んで、プロダクトコード書きに集中し続けられる環境でありたいところ。

psyshはまだあんまり使えていないので、もうちょっと使い込んでみる。

APCをよく使ってたが、最近（PHP5.5など）ではopcacheというもののほうがいいらしい。

いやしかしcomposerは便利だなぁ。
