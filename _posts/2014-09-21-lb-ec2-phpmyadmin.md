---
layout: post
title: "LB+ec2環境にphpmyadminを入れる"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: AWS
published: true
---


MySQLのホストはRDSでも、EC2でも。


## phpmyadminインストール

```sh
# yum -y install phpMyAdmin
```

## config.inc.phpで接続情報を設定

```sh
# vi /usr/share/phpMyAdmin/config.inc.php
```

```php
-$cfg['Servers'][$i]['host']          = ''; // MySQL hostname or IP address
+$cfg['Servers'][$i]['host']          = 'ip-xxx-xxx-xxx-xx.xx-xxxxx-1.compute.internal'; // MySQL hostname or IP address

-$cfg['Servers'][$i]['user']          = '';          // MySQL user
+$cfg['Servers'][$i]['user']          = 'mysqluser';          // MySQL user

-$cfg['Servers'][$i]['password']      = '';          // MySQL password (only needed
+$cfg['Servers'][$i]['password']      = 'mysqluserpassword';          // MySQL password (only needed
```

## conf.d/phpMyAdmin.confのでアクセス制御

```
# vi /etc/httpd/conf.d/phpMyAdmin.conf
```

```
+SetEnvIf X-Forwarded-For "xxx\.xxx\.xxx\.xxx" allowed_access
+SetEnvIf REMOTE_ADDR "xxx\.xxx\.xxx\.xxx" allowed_access

<Directory /usr/share/phpMyAdmin/>
   Order Deny,Allow
   Deny from All
   Allow from 127.0.0.1
+   Allow from env=allowed_access
</Directory>

<Directory /usr/share/phpMyAdmin/scripts/>
   Order Deny,Allow
   Deny from All
   Allow from 127.0.0.1
+   Allow from env=allowed_access
</Directory>
```

## httpdのreload

```
# /etc/init.d/httpd reload
```
