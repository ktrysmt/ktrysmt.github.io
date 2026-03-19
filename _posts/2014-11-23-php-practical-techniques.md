---
layout: post
title: "PHP実践テクニック集"
date: 2014-11-22 15:09:08 +0900
comments: true
categories: PHP
published: true
description: "PHPの実践的なテクニック集。Bing翻訳API、短縮URL生成、プロファイラ設定、アクセラレータ(opcache/APCu/APC)の導入方法をまとめて解説"
redirect_from:
  - /blog/how-to-use-bing-translate-api-for-php/
  - /blog/make-shorturl_bitly/
  - /blog/useful-php-profiler-and-setting/
  - /blog/opcache-apcu-and-apc/
tags:
  - php
  - web
  - performance
  - apache
---

PHPにおける実践的なテクニックをまとめた記事です。外部API連携、プロファイリング、アクセラレータの設定など、開発・運用で役立つ知見を集約しています。

## PHP(CodeIgniter)でBingTranslateAPIを使う

クラス化したので保存しとく。

第三引数にstringがきたらstringを、arrayがきたらarrayを、それぞれ翻訳して返すように加工してある。

`__construct()` はCodeIgniterで使うとき用。

```
class Bing {

  private $client_id = "<YOUR_CLIENT_ID>";
  private $client_secret = "<YOUR_CLIENT_SECRET>";

  function __construct()
  {
    $CI =& get_instance();
    $this->clinet_id       = $CI->config->item("client_id");
    $this->clinet_secret = $CI->config->item("client_secret") );
  }

  public function getTranslatedStringByBingTranslateAPI($from,$to,$object)
  {
    define( 'CLIENT_ID', $this->client_id );
    define( 'CLIENT_SECRET', $this->client_secret );
    define( 'GRANT_TYPE', 'client_credentials' );
    define( 'SCOPE_URL', 'http://api.microsofttranslator.com' );
    define( 'AUTH_URL', 'https://datamarket.accesscontrol.windows.net/v2/OAuth2-13/' );

    $text = (is_array($object)) ? implode("\n",$object) : $object ;

    try
    {
      $accessToken = $this->__getAccessTokens( GRANT_TYPE, SCOPE_URL, CLIENT_ID, CLIENT_SECRET, AUTH_URL );
      $authHeader = 'Authorization: Bearer '.$accessToken;

      $url = 'http://api.microsofttranslator.com/V2/Http.svc/Translate?'
        .http_build_query( compact( 'text', 'from', 'to' ) );

      preg_match("#<string(.+?)>(.+?)</string>#",$this->__request( $url, $authHeader ),$match);
      $response = $match[2];
      if (is_array($object)) $response = explode("\n",$response);
      return $response;
    }
    catch( Exception $e )
    {
      exit(__FILE__.__LINE__." # ".$e);
    }
  }

  private function __getAccessTokens( $grant_type, $scope, $client_id, $client_secret, $auth_url )
  {
    $params = http_build_query(
      compact( 'grant_type', 'scope', 'client_id', 'client_secret' ) );

    $ch = curl_init();
    curl_setopt( $ch, CURLOPT_URL, $auth_url );
    curl_setopt( $ch, CURLOPT_POST, TRUE );
    curl_setopt( $ch, CURLOPT_POSTFIELDS, $params );
    curl_setopt( $ch, CURLOPT_RETURNTRANSFER, TRUE );
    curl_setopt( $ch, CURLOPT_SSL_VERIFYPEER, FALSE );

    $response = curl_exec( $ch );
    if( curl_errno( $ch ) )
    {
      throw new Exception( curl_error( $ch ) );
    }
    curl_close( $ch );

    $json = json_decode( $response );
    if( isset( $json->error ) )
    {
      throw new Exception( $json->error_description );
    }

    return $json->access_token;
  }

    private function __request( $url, $authHeader )
    {
      $ch = curl_init();
      curl_setopt( $ch, CURLOPT_URL, $url );
      curl_setopt( $ch, CURLOPT_HTTPHEADER, array( $authHeader, 'Content-Type: text/xml' ) );
      curl_setopt( $ch, CURLOPT_RETURNTRANSFER, TRUE );
      curl_setopt( $ch, CURLOPT_SSL_VERIFYPEER, FALSE );    $response = curl_exec( $ch );
      if( curl_errno( $ch ) )
      {
        throw new Exception( curl_error( $ch ) );
      }
      curl_close( $ch );    return $response;
    }
}
```

## GoogleとBit.lyで短縮URLを作成するPHPスクリプト

時々使うので自分用にメモ。

### goo.gl

```
  public function make_shorturl_google($url)
  {
     $api_url = "https://www.googleapis.com/urlshortener/v1/url";
     $api_key = "<YOUR API KEY>";
     $curl = curl_init();
     $curl_params = array(
     CURLOPT_URL => $api_url . "?" .
     http_build_query( array( "key" => $api_key ) ),
     CURLOPT_HTTPHEADER => array( "Content-Type: application/json" ),
     CURLOPT_POST => true,
     CURLOPT_POSTFIELDS => json_encode( array( "longUrl" => $url ) ),
     CURLOPT_RETURNTRANSFER => true
     )
          ;
   curl_setopt_array( $curl, $curl_params );
   $result = json_decode( curl_exec( $curl ) );
   return $result->id;
  }
```

### bit.ly

```
  public function make_shorturl_bitly($url)
  {
    $url = "http://api.bit.ly/v3/shorten?"
      ."login=<USERNAME>"
      ."&apiKey=<YOUR_API_KEY>"
      ."&longUrl=".urlencode($url)
      ."&format=xml";

    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
    $data = curl_exec($ch);
    curl_close($ch);

    $data_obj = simplexml_load_string($data);

    if((int)$data_obj->status_code == 200)
    {
      return  (string)$data_obj->data->url;

    }else{
      return FALSE;
    }
  }
```

## PHPのプロファイラとよく使う設定

便利な設定をまとめてメモ。

### 1. Chrome Logger

Chrome拡張は以下からインストール。

- <https://chrome.google.com/webstore/detail/chrome-logger/noaneddfkdjfnfdakjjmocngnfkfehhd>

ライブラリファイルは以下から取得。

- <https://github.com/ccampbell/chromephp>

設定は後述。

### 2. XHProf

#### Install

<http://pecl.php.net/package/xhprof>からtar.gzをダウンロード。

```
yum -y install gcc php php-devel # コンパイルに必要
wget http://pecl.php.net/get/xhprof-0.9.4.tgz
tar xvfz xhprof-0.9.4.tgz
cd xhprof-0.9.4/extension/
phpize
make
make test
make install
```

TEST後にレポートを送るか言われるが一旦無視

```
Do you want to send this report now? [Yns]: n
```

インストール後、php.iniを開いてエクステンション登録

```
extension=xhprof.so
```

その後apache/php-fpmを再起動。

#### deploy

解凍したtar.gz内にxhprof_libおよびxhprof_htmlがあるので参照できるドキュメントルートに配備。

### 3. auto_prepend_file

apacheのconfigかhtaccessに以下を記載。PHPを途中でexit()した場合auto prependされないのだが後述の`register_shutdown_function`を使えば以上終了する場合でも実行してくれる（らしい）。

```
php_value auto_prepend_file /path/to/my_prepend.php
```

その後my_prepend.phpを以下のように記述。

```
<?php
include '/path/to/ChromePhp.php';

// 接続元IPアドレス偽装
#$_SERVER['REMOTE_ADDR'] = 'xxx.xxx.xxx.xxx';

#$_SERVER['HTTP_USER_AGENT'] = 'hogehoge user agent';

// パラメータの強制変更(デバッグ用など)
#$_POST['hoge'] = 'foo';
#$_GET['aaa'] = 'bbb';

// xhprof
function __xhprof_finish() {
  $xhprof_lib = '/path/to/xhprof_lib';
  require_once($xhprof_lib . '/utils/xhprof_lib.php');
  require_once($xhprof_lib . '/utils/xhprof_runs.php');

  $app_name = 'myapp';
  $result = xhprof_disable();
  $runs = new XHProfRuns_Default();
  $run_id = $runs->save_run($result, $app_name);
  $url = 'http://localhost/?run='.$run_id . '&source='.$app_name;
  #error_log($url); // 計測結果の確認URLをerror_logに出力
}

xhprof_enable();
register_shutdown_function('__xhprof_finish');
```

Chrome Loggerは以下のように使う。

```
ChromePhp::log('Hello console!');
ChromePhp::log($_SERVER);
ChromePhp::warn('something went wrong!');
```

### 参考

- <http://blog.asial.co.jp/836>
- <http://quartet-communications.com/info/topics/12238>
- <http://blog.asial.co.jp/1152>
- <http://d.hatena.ne.jp/pasela/20091030/xhprof>
- [auto_prepend_file ・ auto_append_file ディレクティブで自動的にファイルを読み込む | Linuxで自宅サーバ構築](http://linuxserver.jp/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0/php/auto_prepend_file%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%86%E3%82%A3%E3%83%96%E3%81%A8auto_append_file%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%86%E3%82%A3%E3%83%96.php)
- <http://craig.is/writing/chrome-logger>

## PHPアクセラレータまとめ

### php5.5以上の場合 → opcache+APCu

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

### php5.4以下の場合 → APC

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

### おまけ

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

### Reference

- <https://github.com/aws/aws-sdk-php/issues/563>
- <http://blog.hello-world.jp.net/wordpress/2174/>
