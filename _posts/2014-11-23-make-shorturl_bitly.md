---
layout: post
title: "[goo.gl|bit.ly] GoogleとBit.lyで短縮URLを作成するPHPスクリプト"
date: 2014-11-22 15:09:08 +0900
comments: true
categories: PHP
published: true
---

時々使うので自分用にメモ。

## goo.gl

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

## bit.ly

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
