---
layout: post
title: "Apache設定・チューニングまとめ"
date: 2014-05-14 15:09:08 +0900
comments: true
categories: Infrastructure
published: true
description: "Apache本番環境の設定、preforkチューニング、.htaccess、不要モジュール整理、mod_expires/deflateなど運用ノウハウを集約。"
tags:
  - apache
  - security
  - performance
  - linux
  - web
  - infrastructure
redirect_from:
  - /blog/apache-productin-settiong/
  - /blog/apache-tuning/
  - /blog/extra-setting-htaccess/
  - /blog/no-need-LoadModule-apache/
  - /blog/apache-expires-and-deflate/
---

## 本番環境用のApache設定

### confをプロダクション向けに調整するときの確認箇所

+ /etc/httod/conf/httpd.conf
+ iconsディレクティブのindexesを削除
+ vhosts以下のindexesをハイフンに
+ ServerTokens Prod に変更
+ ServerSignature Off  に変更

### Apacheチューニングは以下。

```
  65 PidFile run/httpd.pid
  66
  67 #
  68 # Timeout: The number of seconds before receives and sends time out.
  69 #
  70 Timeout 60
  71
  72 #
  73 # KeepAlive: Whether or not to allow persistent connections (more than
  74 # one request per connection). Set to "Off" to deactivate.
  75 #
  76 KeepAlive On
  77
  78 #
  79 # MaxKeepAliveRequests: The maximum number of requests to allow
  80 # during a persistent connection. Set to 0 to allow an unlimited amount.
  81 # We recommend you leave this number high, for maximum performance.
  82 #
  83 MaxKeepAliveRequests 26
  84
  85 #
  86 # KeepAliveTimeout: Number of seconds to wait for the next request from the
  87 # same client on the same connection.
  88 #
  89 KeepAliveTimeout 5
  90
  91 ##
  92 ## Server-Pool Size Regulation (MPM specific)
  93 ##
  94
  95 # prefork MPM
  96 # StartServers: number of server processes to start
  97 # MinSpareServers: minimum number of server processes which are kept spare
  98 # MaxSpareServers: maximum number of server processes which are kept spare
  99 # ServerLimit: maximum value for MaxClients for the lifetime of the server
 100 # MaxClients: maximum number of server processes allowed to start
 101 # MaxRequestsPerChild: maximum number of requests a server process serves
 102 <IfModule prefork.c>
 103 StartServers       300
 104 MinSpareServers    300
 105 MaxSpareServers    300
 106 ServerLimit        300
 107 MaxClients         300
 108 MaxRequestsPerChild  100
 109 </IfModule>
 110
```

### 不要なバージョン番号やソフトウェア名を隠す

本番環境であれば、隠しておくほうが良いでしょう。

+ LoadModule headers_module modules/mod_headers.so は使用するのでアンコメント。

アンコメントしたらconfの最下行などに以下を記載。

```
Header set Server hogehoge
Header unset X-Powered-By
```

+ Header set Server hogehoge ... サーバー名を上書きして隠蔽（不要？）
+ Header unset X-Powered-By ... 不要は宣言はunsetで消す

## Apache preforkのチューニング

### preforkの場合

- apacheの親プロセスが消費するメモリ数とシステムが消費するメモリ数を把握する
- apacheの１子プロセスあたりで消費するメモリ数を把握する
- 物理メモリ数から、apacheの親プロセスが消費するメモリ数＋システムが消費するメモリ数を引く
- 引いた残りを１子プロセスあたりで消費するメモリ数で割る
- 割った数を、MacClientの設定値とする

### ポイント

プロセスの上げ下げにもリソースは消費されるので、

```
StartServers
MinSpareServers
MaxSpareServers
ServerLimit
MaxClients
```

すべてMaxClientsの数値と同値にするのが良い。

### 子プロセスの非共有メモリ消費量算出

こちらのperlスクリプトを使ってます。感謝。

Apacheとかforkしたプロセスのメモリチューニングに関するメモとスクリプト - [http://hirobanex.net/article/2013/10/1381407737](http://hirobanex.net/article/2013/10/1381407737)

ここまで書いておいてなんですがPHP使うならnginxの方がいいと思いはじめてる今日この頃。

## 便利な.htaccessの書き方いろいろ

ときどき使うのでとりとめなくまとめ

### メンテナンス対応のときの、メンテ画面リダイレクト処理

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

### 5.2から5.3へバージョンアップした時は以下を差し込んでおくとしのげる

```
php_flag allow_call_time_pass_reference on
```

### LB経由（X-Forwaded-For）でアクセス制限

変数を使えばOK

```
SetEnvIf X-Forwarded-For "^xxx\.xxx\.xxx\.xxx$" allowed_access=yes
SetEnvIf X-Forwarded-For "^yy\.yy\.yy\.yy$" allowed_access=yes
SetEnvIf REMOTE_ADDR "^zz\.zz\.zz\.zz$" allowed_access=yes
order deny,allow
deny from all
allow from env=allowed_access
```

### 特定のファイルにだけアクセス制限

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

### UserAgentやRequestURIで分岐

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

### 特定のIP・ホストにだけアクセスを許可する

```
order deny,allow
deny from all
allow from xx.xx.xx.xx
```

### 参考

+ <http://d.hatena.ne.jp/camelmasa/20080626/1214451268>

## だいたいコメントアウトするApacheモジュールたち

そもそもApacheをあまり使わなくなってきているが

```
vim /etc/httpd/conf/httpd.conf
```

```sh
 154 #
 155 LoadModule auth_basic_module modules/mod_auth_basic.so
 156 LoadModule auth_digest_module modules/mod_auth_digest.so
 157 LoadModule authn_file_module modules/mod_authn_file.so
 158 #LoadModule authn_alias_module modules/mod_authn_alias.so
 159 #LoadModule authn_anon_module modules/mod_authn_anon.so
 160 #LoadModule authn_dbm_module modules/mod_authn_dbm.so
 161 LoadModule authn_default_module modules/mod_authn_default.so
 162 LoadModule authz_host_module modules/mod_authz_host.so
 163 LoadModule authz_user_module modules/mod_authz_user.so
 164 #LoadModule authz_owner_module modules/mod_authz_owner.so
 165 #LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
 166 #LoadModule authz_dbm_module modules/mod_authz_dbm.so
 167 LoadModule authz_default_module modules/mod_authz_default.so
 168 #LoadModule ldap_module modules/mod_ldap.so
 169 #LoadModule authnz_ldap_module modules/mod_authnz_ldap.so
 170 LoadModule include_module modules/mod_include.so
 171 LoadModule log_config_module modules/mod_log_config.so
 172 LoadModule logio_module modules/mod_logio.so
 173 LoadModule env_module modules/mod_env.so
 174 #LoadModule ext_filter_module modules/mod_ext_filter.so
 175 LoadModule mime_magic_module modules/mod_mime_magic.so
 176 LoadModule expires_module modules/mod_expires.so
 177 LoadModule deflate_module modules/mod_deflate.so
 178 LoadModule headers_module modules/mod_headers.so
 179 #LoadModule usertrack_module modules/mod_usertrack.so
 180 LoadModule setenvif_module modules/mod_setenvif.so
 181 LoadModule mime_module modules/mod_mime.so
 182 #LoadModule dav_module modules/mod_dav.so
 183 LoadModule status_module modules/mod_status.so
 184 LoadModule autoindex_module modules/mod_autoindex.so
 185 LoadModule info_module modules/mod_info.so
 186 #LoadModule dav_fs_module modules/mod_dav_fs.so
 187 LoadModule vhost_alias_module modules/mod_vhost_alias.so
 188 LoadModule negotiation_module modules/mod_negotiation.so
 189 LoadModule dir_module modules/mod_dir.so
 190 #LoadModule actions_module modules/mod_actions.so
 191 #LoadModule speling_module modules/mod_speling.so
 192 #LoadModule userdir_module modules/mod_userdir.so
 193 LoadModule alias_module modules/mod_alias.so
 194 LoadModule rewrite_module modules/mod_rewrite.so
 195 #LoadModule proxy_module modules/mod_proxy.so
 196 #LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
 197 #LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
 198 #LoadModule proxy_http_module modules/mod_proxy_http.so
 199 #LoadModule proxy_connect_module modules/mod_proxy_connect.so
 200 #LoadModule cache_module modules/mod_cache.so
 201 LoadModule suexec_module modules/mod_suexec.so
 202 #LoadModule disk_cache_module modules/mod_disk_cache.so
 203 #LoadModule file_cache_module modules/mod_file_cache.so
 204 #LoadModule mem_cache_module modules/mod_mem_cache.so
 205 LoadModule cgi_module modules/mod_cgi.so
 206 #LoadModule version_module modules/mod_version.so
 207 LoadModule ssl_module modules/mod_ssl.so
```

## httpdのパフォーマンス改善（expires, deflate）

Cloudflare使うかとかそもそもhttpdやめればとかはともかく必要なケースもあるのでコピペ用にまとめ。

```
# mod_expire
ExpiresActive On
ExpiresByType image/gif "access plus 3 days"
ExpiresByType image/jpeg "access plus 3 days"
ExpiresByType image/png "access plus 3 days"
ExpiresByType text/css "access plus 3 days"
ExpiresByType application/x-javascript "access plus 3 days"

# mod_deflate
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/rdf+xml
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/css

```
