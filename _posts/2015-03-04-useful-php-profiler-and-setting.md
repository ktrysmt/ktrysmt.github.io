---
layout: post
title: "PHPのプロファイラとよく使う設定"
date: 2015-03-03 15:09:08 +0900
comments: true
categories: PHP
published: true
---

便利な設定をまとめてメモ。


## 1. Chrome Logger

Chrome拡張は以下からインストール。

- <https://chrome.google.com/webstore/detail/chrome-logger/noaneddfkdjfnfdakjjmocngnfkfehhd>

ライブラリファイルは以下から取得。

- <https://github.com/ccampbell/chromephp>

設定は後述。

## 2. XHProf

### Install

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

### deploy

解凍したtar.gz内にxhprof_libおよびxhprof_htmlがあるので参照できるドキュメントルートに配備。

## 3. auto_prepend_file

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

## 参考

- <http://blog.asial.co.jp/836>
- <http://quartet-communications.com/info/topics/12238>
- <http://blog.asial.co.jp/1152>
- <http://d.hatena.ne.jp/pasela/20091030/xhprof>
- [auto_prepend_file ・ auto_append_file ディレクティブで自動的にファイルを読み込む | Linuxで自宅サーバ構築](http://linuxserver.jp/%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0/php/auto_prepend_file%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%86%E3%82%A3%E3%83%96%E3%81%A8auto_append_file%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%86%E3%82%A3%E3%83%96.php)
- <http://craig.is/writing/chrome-logger>
