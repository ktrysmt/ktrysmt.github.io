---
layout: post
title: "本番環境用のApacheの設定"
date: 2014-05-14 15:09:08 +0900
comments: true
categories: Apache
published: true
---

## Apacheのconfをプロダクション向けに調整するときの確認箇所

+ /etc/httod/conf/httpd.conf
+ iconsディレクティブのindexesを削除
+ vhosts以下のindexesをハイフンに
+ ServerTokens Prod に変更
+ ServerSignature Off  に変更

## Apacheチューニングは以下。

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

## 不要なバージョン番号やソフトウェア名を隠す
本番環境であれば、隠しておくほうが良いでしょう。

+ LoadModule headers_module modules/mod_headers.so は使用するのでアンコメント。

アンコメントしたらconfの最下行などに以下を記載。

```
Header set Server hogehoge
Header unset X-Powered-By
```

+ Header set Server hogehoge … サーバー名を上書きして隠蔽（不要？）
+ Header unset X-Powered-By … 不要は宣言はunsetで消す

