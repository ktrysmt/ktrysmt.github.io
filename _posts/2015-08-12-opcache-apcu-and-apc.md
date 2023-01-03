---
layout: post
title: "PHPアクセラレータまとめ"
date: 2015-07-01 15:09:08 +0900
comments: true
categories: PHP
published: true
---

## php5.5以上の場合 → opcache+APCu

```
# yum -y install php55-opcache (amzn-main)
# yum -y install php55-opcache --enablerepo=remi-php55 (remi-php55)
# pecl install APCu-beta

# vim /etc/php.d/opcache.ini #ファイルキャッシュを不可(=0)にするか2秒に変更
opcache.memory_consumption=256 #共有メモリとして256MB設定
opcache.revalidate_freq=0 # 開発環境では0
opcache.revalidate_freq=2 # 本番環境では2にする

# vim /etc/php.d/apcu.ini
extension=apcu.so
apc.enabled = 1
apc.shm_size=256M #共有メモリとして256MB設定
apc.ttl = 3600
apc.user_ttl = 3600
apc.gc_ttl = 7200
apc.stat = 1

# /etc/init.d/httpd restart #apache再起動で反映
```

## php5.4以下の場合 → APC

```
# yum install php-pecl-apc

# vi /etc/php.d/apc.ini
extension = apc.so
apc.enabled = 1
apc.shm_size = 128M
apc.ttl = 3600
apc.user_ttl = 3600
apc.gc_ttl = 7200
apc.stat = 1

# /etc/init.d/httpd restart #apache再起動で反映
```

## おまけ

こういったキャッシュ系と圧縮済みのphar とは相性が悪い。

opcacheなら

```
# vim /etc/php.d/opcache-default.blacklist

/path/to/file/aws.phar
```

APCなら

```
# vim /etc/php.d/apc.ini

apc.filters="^phar://"
```

と除外リストに入れておくといい。

よくつまづくのはawsが提供しているawd-sdk-phpがコールした時にキャッシュのせいでうまく参照できないエラーが出るトラブル。

## Reference

- <https://github.com/aws/aws-sdk-php/issues/563>
- <http://blog.hello-world.jp.net/wordpress/2174/>
