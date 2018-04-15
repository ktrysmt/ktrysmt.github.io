---
layout: post
title: "PHP(CodeIgniter)でBingTranslateAPIを使う"
date: 2014-11-22 15:09:08 +0900
comments: true
categories: PHP
published: true
---

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
