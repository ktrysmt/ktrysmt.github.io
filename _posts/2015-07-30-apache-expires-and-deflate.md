---
layout: post
title: "httpdのパフォーマンスをさっさと改善(expires,deflate)"
date: 2015-07-10 15:09:08 +0900
comments: true
categories: Apache
published: true
---

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
