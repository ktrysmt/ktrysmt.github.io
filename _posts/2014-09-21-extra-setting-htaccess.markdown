---
layout: post
title: "便利な.htaccessの書き方いろいろ"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: Apache
published: true
---

ときどき使うのでとりとめなくまとめ

## メンテナンス対応のときの、メンテ画面リダイレクト処理

```
ErrorDocument 503 /mainte.html
RewriteEngine on
RewriteCond %{REMOTE_ADDR} !^192.68.
RewriteCond %{REMOTE_ADDR} !^xx.xx.xx.xx$
RewriteCond %{HTTP:X-Forwarded-For} !^192.168.
RewriteCond %{HTTP:X-Forwarded-For} !^xx.xx.xx.xx$
RewriteCond %{REQUEST_URI} !(^/image/)
RewriteCond %{REQUEST_URI} !(^/css/)
RewriteCond %{REQUEST_URI} !(^/js/)
RewriteCond %{REQUEST_URI} !/mainte.html
RewriteCond %{TIME} >20140310020000
RewriteCond %{TIME} <20140310060000
RewriteRule ^.*$ - [R=503,L]
```

## 5.2から5.3へバージョンアップした時は以下を差し込んでおくとしのげる

```
php_flag allow_call_time_pass_reference on
```


## LB経由（X-Forwaded-For）でアクセス制限
変数を使えばOK

```
SetEnvIf X-Forwarded-For "^xxx\.xxx\.xxx\.xxx$" allowed_access=yes
SetEnvIf X-Forwarded-For "^yy\.yy\.yy\.yy$" allowed_access=yes
SetEnvIf REMOTE_ADDR "^zz\.zz\.zz\.zz$" allowed_access=yes
order deny,allow
deny from all
allow from env=allowed_access
```


## 特定のファイルにだけアクセス制限
adminerが便利で、よく使います。

前述の`allowed_access`変数も使う。

```
<Files ~ "\.(html|php)$">
  RewriteEngine Off
</Files>
<Files ~ "adminer\.php$">
order deny,allow
deny from all
allow from env=allowed_access
allow from xx.xx.xx.xx
allow from yyy.yyy.
</Files>
```

## UserAgentやRequestURIで分岐

```
RewriteEngine On
RewriteBase /

RewriteCond %{HTTP_USER_AGENT} (DoCoMo|KDDI|Softbank) [NC]
RewriteCond %{HTTP_HOST} ^(www.example.com|smart.example.com)
RewriteCond %{REQUEST_URI} !(error|sample)
RewriteRule ^(.*)$ https://mobile.example.com/$1 [QSA,R=301,L]

RewriteCond %{HTTP_USER_AGENT} (iPod|Android|iPhone|iPad|blackberry|webOS|Phone) [NC]
RewriteCond %{HTTP_HOST} ^(www.example.com|mobile.example.com)
RewriteCond %{REQUEST_URI} !(error|sample)
RewriteRule ^(.*)$ https://smart.example.com/$1 [QSA,R=301,L]

RewriteCond %{ENV:allowed_access} !yes
RewriteCond %{REQUEST_URI} ^/$
RewriteRule .* ./index.php [L]
```

## 特定のIP・ホストにだけアクセスを許可する

```
order deny,allow
deny from all
allow from xx.xx.xx.xx
```

## 参考

+ <http://d.hatena.ne.jp/camelmasa/20080626/1214451268>
