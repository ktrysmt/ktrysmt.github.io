---
layout: post
title: "apache+mod_php を nginx+php-fpmに移行 (+ TCP Connection & File Descriptor)"
date: 2014-06-19 15:09:08 +0900
comments: true
categories: PHP
published: true
---


## もくじ

- Install Dependency Libraries.
- TCP Connection & Backlog ... `/etc/sysctl.conf`
- File Descriptor ... `/etc/initscript`
- Nginx 全体設定 ... `/etc/nginx/nginx.conf` （include活用して可読性を高めるとなおよい）
- Nginx 各個Server設定 ... `/etc/nginx/conf.d/*.conf`
- PHP-FPM 全体設定 ... `/etc/php-fpm.conf`
- PHP-FPM 各個設定 ... `/etc/php-fpm.d/www.conf`
- Apacheバーチャルホスト、htaccessの移行
- おまけ１：NginxでApacheのバーチャルドキュメントルートもどき
- おまけ２：サブドメイン方式ではなくサブディレクトリで運用

これらのチューニングを一気にやる、保存版。

※おことわり

- OSはCentOS 6 64bitです。
- PHPアクセラレータの利用は長くなるので割愛、<http://keidrip.github.io/blog/opcache-apcu-and-apc/>を参照のこと。
- PHP >= 5.5推奨で、epel,remi(remi-php55等)を使うこと前提になっています、3rdPartyのインストールは<http://keidrip.github.io/blog/add-3rd-repositories-to-centos-6/>を参照。










## Install Dependency Libraries.

nginxとphp-fpmをインストール。

```
# yum -y install nginx php-fpm --enablerepo=epel,remi,remi-php55
# chkconfig php-fpm on
# chkconfig nginx on
```

## TCP Connection & Backlog

### sysctl操作

```
# cat /proc/sys/fs/file-nr #現在のOS全体のファイルディスクリプタ上限を確認
736     0       57677
# ulimit -n #現在のプロセスごとのファイルディスクリプタ上限を確認
1024
```

```
# vim /etc/sysctl.conf #以下の内容を追記して設定変更
fs.file-max = 794573
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_fin_timeout = 20
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.ip_local_port_range = 1024    65535
net.core.somaxconn = 10240
```

```
# sysctl -p #設定の反映
```

### 変更できたかチェック

```
# cat /proc/sys/fs/file-nr #上限増加を確認
736     0       794573
```

なおnet.ipv4.tcp_tw_recycleは一応0にしておく。将来1にするかもしれないけど。








## File Descriptor

```
# echo 'ulimit -n 65536
eval exec "$4"' > /etc/initscript

# reboot #マシン再起動で反映

# ulimit -n # 再起動後、上限増加を確認
65536
```

デーモンプロセスのファイルディスクリプタも以下で確認できる。

```
# cat /proc/`pgrep ＜プロセス名＞ | head -1`/limits | grep 'open files'
```

php-fpmやnginxなら以下。

```
# cat /proc/`pgrep nginx | head -1`/limits | grep 'open files'
# cat /proc/`pgrep php-fpm | head -1`/limits | grep 'open files'
```

### （補足）/etc/security/limits.conf

/etc/initscriptの設定だけで良いようだが一応メモ。

```
# vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
```








## Nginx 全体設定

worker_rlimit_nofileを65536にするのはちと不安なので10240にしておく（マシンのCore数が多ければ増やしてもいいのかも？）。
アクセスログはPHPの処理にだけ吐き出すようにするので、全体設定ではoffにしておく。

```
# mkdir /var/tmp/nginx
# mkdir -p /var/cache/nginx/{proxy,fastcgi}
# chown -R nginx:nginx /var/tmp/nginx
# chown -R nginx:nginx /var/cache/nginx
# chown -R nginx /var/lib/php/session
# chown nginx /var/log/php-fpm
# rm /etc/nginx/conf.d/default.conf; # defaults.confは不要なので消す
```

```
# vim /etc/nginx/nginx.conf

worker_processes  1; #CPUコア数によって変更
worker_rlimit_nofile 10240;

events {
    worker_connections  1024;
    multi_accept on;
    use epoll;
}

http {

    # 基本設定
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $upstream_cache_status - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log off;

    # CacheDir設定
    # ※levels,keys_zoneサイズ,max_size,inactiveはアプリによって変更
    proxy_cache_path   /var/cache/nginx/proxy levels=1:2 keys_zone=PROXY:15m max_size=1000m inactive=1d;
    proxy_temp_path    /var/tmp/nginx;
    # fastcgi cacheを使う場合は以下をアンコメント
    #fastcgi_cache_path /var/cache/nginx/fastcgi levels=1:2 keys_zone=FASTCGI:15m max_size=1000m inactive=1d;
    #fastcgi_cache_key  "$scheme$request_method$host$request_uri";

    # Signature設定
    sendfile       on;
    tcp_nopush     on;
    server_tokens  off;
    keepalive_timeout  0;
    port_in_redirect off;
    send_timeout   30;
    reset_timedout_connection on;
    tcp_nodelay    on;

    # gzip設定
    # ※CPU負荷をみて調整すること
    gzip             on;
    gzip_static    on;
    #gunzip on; # モジュールがないと使えないので注意
    #gzip_http_version 1.0; #不安定
    gzip_types        text/plain
                      text/xml
                      text/css
                      text/javascript
                      application/xml
                      application/xhtml+xml
                      application/rss+xml
                      application/atom_xml
                      application/javascript
                      application/x-javascript
                      application/json
                      application/x-httpd-php;
    gzip_disable      "MSIE [1-6]\.";
    gzip_disable      "Mozilla/4";
    gzip_comp_level   6;
    gzip_proxied      any;
    gzip_vary         on;
    gzip_buffers      4 8k;
    gzip_min_length   1000;

    # open_file_cache設定
    # ※中規模以上なら有効にする
    #open_file_cache max=10000 inactive=20s; # 場合によってはmax=100000
    #open_file_cache_valid 30s;
    #open_file_cache_min_uses 2;
    #open_file_cache_errors on;

    include       /etc/nginx/conf.d/*;
}
```






## Nginx 各個Server設定

だいたい以下がテンプレ。

```
upstream static {
  server localhost:8080;
}
server { # upstream static用
  listen       8080;
  server_name localhost;
  root   /var/www/example.com/public;
}

server {
  listen       80;
  server_name  example.com;
  #root         /var/www/example.com/public/;
  #set          $do_not_fastcgi_cache 1; # デフォルトで全体キャッシュなら1を有効に

  location / {
      index index.php index.html index.htm;
      if ($request_uri ~* "(rss|sitemap.xml)$") {
            break;
          }
      if (!-f $request_filename) {
            rewrite ^(.+)$ /index.php?$1 last;
            break;
          }
      try_files $uri $uri/ @staticsite;
      expires 30d;
  }

  location ~ \.php$ {

     #if ($args ~* /admin/) { set $do_not_fastcgi_cache 0;} #特定のディレクトリだけキャッシュ無効にするならアンコメン

     access_log     /var/log/nginx/access.log  main;
     fastcgi_read_timeout 30;
     fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock; # unixドメインソケットを使う（PHP-FPMと連携）
     fastcgi_index  index.php;
     fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
     include        fastcgi_params;

     # $do_not_fastcgi_cache変数を使う場合アンコメント
     #fastcgi_cache_bypass $do_not_fastcgi_cache;
     #fastcgi_no_cache $do_not_fastcgi_cache;

     # cache機構を使う場合アンコメント
     #fastcgi_cache FASTCGI;
     #fastcgi_cache_valid  200 6h;
     #fastcgi_cache_valid  404 1d;
   }

    location @staticsite {

    proxy_pass http://static;
    proxy_cache PROXY;
    #proxy_ignore_headers X-Accel-Redirect X-Accel-Expires Cache-Control Expires Set-Cookie; # CookieやExpires有効時はCacheを無効にする
    add_header X-Cache-Status $upstream_cache_status;
    proxy_cache_key "$scheme://$host$request_uri$is_args$args";
    proxy_cache_valid  200 302  7d;
    proxy_cache_valid  404 7d;

    # LB経由の場合はx-forwarded-forを使用
    #proxy_set_header X-Real-IP  $remote_addr;
    #proxy_set_header Host $host;
    #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```




## PHP-FPM 全体設定

```
# vim /etc/php-fpm.conf

[global]
pid = /var/run/php-fpm.pid
error_log = /var/log/php/php-fpm.log
log_level = notice
emergency_restart_threshold = 0
emergency_restart_interval = 0
process_control_timeout = 0
daemonize = yes

process_control_timeout = 10
emergency_restart_interval = 60
emergency_restart_threshold = 10
```





## PHP-FPM 各個設定

チューニングはまだまだ不勉強なので、稼働させてみてから調整。

```
# vim /etc/php-fpm.d/www.conf

[www]
listen = /var/run/php-fpm/php-fpm.sock;
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
listen.backlog = -1

user = nginx
group = nginx

; メモリが少ないか極端にスペックの低いサーバなら、dynamicにして影響を最小限に
pm = dynamic
pm.max_children =  25
pm.start_servers = 1
pm.min_spare_servers = 1
pm.max_spare_servers = 3
pm.max_requests = 50

; CPUに不安があるか余裕があるならstaticにしてメモリを予め確保しておく
pm = static
pm.max_children = 20
pm.start_servers = 5
pm.min_spare_servers = 2
pm.max_spare_servers = 8
pm.max_requests = 100

slowlog = /var/log/php-fpm/slowlog
;request_slowlog_timeout = 60 # アプリの内容によって調整
request_terminate_timeout= 30
rlimit_files = 10240 # nginxと揃えとく
```









## Apacheバーチャルホスト、htaccessの移行

apacheのmod_rewriteをnginxのrewriteに置き換える。

```
# cat .htaccess
RewriteEngine on
RewriteCond $1 !^(index\.php|images|javascripts|stylesheets|robots\.txt|favicon\.ico|css|js)
RewriteRule ^(.*)$ /index.php/$1 [QSA,L]
```

この場合は例えば以下のように置き換え。

```
# vim /etc/nginx/conf.d/virtual.conf

server {
  listen       80;
  server_name  example.dev;
  root         /var/www/vhosts/example;
  #set          $do_not_fastcgi_cache 1;

  location / {
    index index.php index.html index.htm;
    if ($request_uri ~* "(rss|sitemap.xml)$") {
      break;
    }
    if (!-f $request_filename) {
      rewrite ^(.+)$ /index.php?$1 last;
      break;
    }
    if (-f $request_filename) {
      expires 30d;
    }
    proxy_cache PROXY;
    #proxy_ignore_headers Cache-Control; # proxy_pass使う場合有効に。
    proxy_cache_key "$scheme://$host$request_uri$is_args$args";
    proxy_cache_valid  200 302  7d;
    proxy_cache_valid  404 7d;
  }
  location ~ \.php$ {

    #if ($args ~* /d/) { set $do_not_fastcgi_cache 0;}

    access_log     /var/log/nginx/access.log  main;
    fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;

    #fastcgi_cache_bypass $do_not_fastcgi_cache;
    #fastcgi_no_cache $do_not_fastcgi_cache;
    #fastcgi_cache FASTCGI;
    #fastcgi_cache_valid  200 1h;
    #fastcgi_cache_valid  404 1d;
  }
}
```

### シンタックスチェックしてapacheからnginxにバトンタッチ

```
# /etc/init.d/nginx configtest
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# /etc/init.d/php-fpm start;/etc/init.d/httpd stop;/etc/init.d/nginx start
```














## おまけ１：NginxでApacheのバーチャルドキュメントルートもどき

nginx.confとvirtula.confを修正

```
# vim /etc/nginx/nginx.conf #サブドメイン名が長くなるなら数字を大きめに設定

http {
  server_names_hash_bucket_size  64;
  server_names_hash_max_size    512;
（以下略）
```

```
# vim /etc/nginx/conf.d/virtual.conf

server {

  listen 80;
  server_name ~^(.*)\.example.com$;
  set $subdomain $1;
  #set $do_not_fastcgi_cache 1;

  if (!-d /var/www/vhosts/$subdomain) {
    rewrite . http://example.com/ redirect;
  }

  location / {
    index index.php;
    root /var/www/vhosts/$subdomain/;

    if ($request_uri ~* "(rss|sitemap.xml)$") {
      break;
    }
    if (!-f $request_filename) {
      rewrite ^(.+)$ /index.php?$1 last;
    }
    if (-f $request_filename) {
      expires 30d;
      break;
    }
    proxy_cache PROXY;
    proxy_cache_key "$scheme://$host$request_uri$is_args$args";
    proxy_cache_valid  200 302  7d;
    proxy_cache_valid  404 7d;
  }
  location ~ \.php$ {
    #access_log     /var/log/nginx/access.log  main;
    fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_param SCRIPT_FILENAME /var/www/vhosts/$subdomain/$fastcgi_script_name;
    include fastcgi_params;

    #fastcgi_cache_bypass $do_not_fastcgi_cache;
    #fastcgi_no_cache $do_not_fastcgi_cache;
    #fastcgi_cache FASTCGI;
    #fastcgi_cache_valid  200 1h;
    #fastcgi_cache_valid  404 1d;
  }
}
```





## おまけ２：サブドメイン方式ではなくサブディレクトリで運用

以下は特にCodeIgniterの場合。

```
server {
  listen       80;
  server_name  sample.dev;
  root         /var/www/sample;

  location / {
    index index.php index.html index.htm;
    if (-f $request_filename) {
      expires 30d;
      break;
    }
    if (!-f $request_filename) {
      rewrite ^/(.+?)/(.*)$ /$1/index.php?$2 last;
    }
  }

  location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
  }
}
```
