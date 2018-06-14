---
layout: post
title: "Nginxの名前解決についてまとめ"
date: 2017-09-30
comments: true
categories: "Nginx"
published: true
use_toc: true
description: "Nginxの名前解決にはいくつか特徴があり，把握しておかないとまれにトラブルに見舞われることがある（あった）のでまとめました。" 
---

Microservicesが市民権を得てきた昨今，特にAWSのELB等をInternalなロードバランサとして使うことも当たり前になり，ハマるひとも多かったのではないでしょうか。私も漏れなくハマりまして，以前遭遇した時はNginxのバグではないかと疑っていたのですが，よく調べたらどうもこの辺りの挙動は仕様のようです。

以下具体的に述べます。

## 1. Nginxは通常，起動時に行った名前解決の結果を保持し続ける

まずこの仕様を知っていないとトラブルのときに確実に慌てると思われます…。
この仕様は公式Blogで説明されています。

* <https://www.nginx.com/blog/dns-service-discovery-nginx-plus/>

> * If the domain name can't be resolved, NGINX fails to start or reload its configuration.
> * NGINX caches the DNS records until the next restart or configuration reload, ignoring the records' TTL values.

再度名前を引かせるためにはNginxをreload, stop/start, あるいはrestartをする必要があるということになります。

## 2. proxy_passにURLをsetした変数を渡すと名前解決にresolverを使うようになる

`1.`の挙動により問題になるのが，proxy_passの宛先にELBなど動的にIPが変わるURLを指定したい場合です。

```nginx
location / {
    proxy_pass "http://LoadBalancer-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com";
}
```

ELBは不定期にIPが変わるため，この書き方ではいずれどこかのタイミングでNetwork unreachableになってしまいます。

公式のproxy_passでは，以下のような説明があります。

<http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass>

> Parameter value can contain variables. In this case, if an address is specified as a domain name, the name is searched among the described server groups, and, if not found, is determined using a resolver.

そこで以下のような書き方をします。

```nginx
http {
    resolver 169.254.169.253;

    location / {
        set $url "http://LoadBalancer-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com";
        proxy_pass $url;    
    }
}
```

location内だけでこういうふうにもかけます。

```nginx
location / {
    resolver 8.8.8.8 valid=5s;
    set $url "http://LoadBalancer-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com";
    proxy_pass $url;
}
```

http|server|locationコンテキストのいずれかにてsetでurlを変数定義し，かつresolverを定義します。
前者はhttpコンテキストにおいてsetが使われる場所の名前解決をTTLのタイミングで行ってくれます。
後者の場合はlocationコンテキストでのみresolverが有効となり，TTL関係なく5秒おきに名前解決します。

なおここで使っている169.254.169.253はAWSが提供しているDNSサーバで，8.8.8.8はGoogleが提供しているそれです（`8.8.8.8 valid=5s`は，あまりお行儀が良くないですが…）。

AWS ELBの文脈で言うと，VPCを使っているならば`CIDR + 2`のIPがVPCの内部DNSとして割り当てられるので，そちらを指定してもいいですね。

## 3. upstreamでもresolverを使いたい（が，そうやすやすと使わせてくれない）

`2.`の方法で万事解決かと思いきやこのテクニックはupstreamと組み合わせて使うことができません。
upstream内ではsetを使えない仕様であるためです。

対策としては3つあります。

* 有償版のNginx Plusを使う
* 3rd Partyモジュールを使う
* Unixドメインソケットを経由して名前解決とupstreamの仕事を分割する

### 有償版のNginx Plusを使う

これが一番手っ取り早いですが，できることなら選びたくはないですよね…。
有償版を使うとresolveフラグが使えるようになるので，以下のような記述が可能になります。

```nginx
http {
    resolver 169.254.169.253;

    upstream backend {
        server LoadBalancer-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com resolve;
    }
}
```

### 3rd Partyモジュールを使う

よく言及されているのは[GUI/nginx-upstream-dynamic-servers](https://github.com/GUI/nginx-upstream-dynamic-servers)を使う方法ですが，長らくメンテされていないようです。readmeにも以下の記述があり，最近のNginxでは動かない可能性が高いです。

> Tested with nginx 1.6, 1.7, 1.8, 1.9.

なので，使わないほうが無難かと思われます。

2017/09/30時点であれば，公式Wikiでも紹介のあるngx_upstream_jdomainを使うのが良さそうです。

* <https://www.nginx.com/resources/wiki/modules/domain_resolve/>
* <https://github.com/wdaike/ngx_upstream_jdomain>

これを含めてビルドすることで，以下のような記述が可能になります。

```nginx
http {
    resolver 8.8.8.8;
    resolver_timeout 10s;

    upstream backend {
        jdomain www.baidu.com;
        #keepalive 10;
    }
    server {
        listen       8080;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

jdomainではupstreamコンテキスト内において他にも以下のディレクティブを定義できるようです。

| Directive  | Desc                                     |
| ---------- | ---------------------------------------- |
| port:      | Backend’s listening port.                |
| interval:  | How many seconds to resolve domain name. |
| max_ips:   | IP buffer size.                          |
| retry_off: | Do not retry if one IP fails.            |

プロダクションに投入するとなると信頼性やパフォーマンス面が少し気になるところですが，一応公式Wikiで紹介されているものですし，うまくハマれば要求は満たせそうです。

### Unixドメインソケットを経由して名前解決とupstreamの仕事を分割する

[Qiita](https://qiita.com/minamijoyo/items/183e51a28a3a9d79182f)で見つけて目からウロコだったやり方です。
詳細はリンク先に譲るとして，コードだけ抜粋です。

```nginx
http {
    ...
    resolver 10.9.0.2 valid=5s;

    upstream backend {
        server unix:/var/run/nginx_backend1.sock weight=9 max_fails=1 fail_timeout=20s;
        server unix:/var/run/nginx_backend2.sock weight=1;
    }

    server {
        listen       8080;
        server_name  example.com;
        proxy_next_upstream error timeout http_502 http_503 http_504;;
        ...

        location / {
            proxy_pass  http://backend;
        }
    }

    server {
        listen       unix:/var/run/nginx_backend1.sock;
        server_name  example.com;
        ...

        set $lb_backend1 "internal-lb-backend1-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com";

        location / {
            proxy_pass  http://$lb_backend1:80;
        }
    }

    server {
        listen       unix:/var/run/nginx_backend2.sock;
        server_name  example.com;
        ... 

        set $lb_backend2 "internal-lb-backend2-XXXXXXXXX.ap-northeast-1.elb.amazonaws.com";

        location / {
            proxy_pass  http://$lb_backend2:80;
        }
    }
}
```

upstreamに定義したいproxy先の数だけserverコンテキストを書く必要があり少し冗長ですが，
全体的に簡潔で見通しの良いコードにおさまりそうです。
流量によってはulimitなどでファイルディスクリプタまわりを調整してあげると良さそうです。

## 参考

* <https://www.nginx.com/blog/dns-service-discovery-nginx-plus/>
* <https://www.nginx.com/resources/wiki/modules/domain_resolve/>
* <https://github.com/wdaike/ngx_upstream_jdomain>
* <https://github.com/GUI/nginx-upstream-dynamic-servers>
* <https://qiita.com/minamijoyo/items/183e51a28a3a9d79182f>
