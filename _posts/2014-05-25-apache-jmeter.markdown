---
layout: post
title: "Apache JMeterの使い方"
date: 2014-05-25 15:09:08 +0900
comments: true
categories: Apache
published: true
---

[今から３分で jmeter の使い方を身に付ける　　（負荷テスト入門） - 主に言語とシステム開発に関して][1].

 [1]: http://d.hatena.ne.jp/language_and_engineering/20081014/1223905380

こちらの記事を参考にセットアップしつつ実際に使ってみた手順をメモ。




## 全体の流れ
- JavaのインストールとJMeterのダウンロード
- GUIでJMeterの設定を行う（今回はWindows環境で解説）
- 負荷テスト実行サーバに設定ファイルの.jmxをアップロード
- 負荷テスト実行サーバにjavaをインストールし、JMeterのtar.gzをダウンロード、解凍
- JMeterを実行




## JavaのインストールとJMeterのダウンロード

### Java JREをダウンロード

とにもかくにもJMeterの動作にはJavaが必要。ダウンロードはこちら。<http://java.com/ja/download/>

### Apache JMeterをダウンロード

Apache JMeter本体はこちら。<http://jmeter.apache.org/download_jmeter.cgi>





## GUIでJMeterの設定を行う（今回はWindows環境で解説）

### ダウンロードしたzipを解凍し、bin/jmeter.batを実行

実行すると以下の様な画面が開く。

![](/images/article/jmeter-1.png)

### 「テスト計画」配下に「スレッドグループ」を追加

テスト計画を右クリックするとメニューが表示される。

```
> 追加＞Threads(Users)＞スレッドグループ
```

を選択。

スレッド数がRamp-Up期間(秒)の間に、ループ回数分実行されるということらしい。
「Delay Thread creation until needed」にチェックを入れると、必要になったタイミングでスレッド生成がされるようになるので、
チェックを入れるほうが良い。
入れないと実行直後に一気にスレッド生成をしてしまって、
負荷テスト実行サーバ側がハングアウトしてしまう可能性があるかもしれない。

![](/images/article/jmeter-2.png)

### 「スレッドグループ」配下に「HTTPリクエスト」を作成

次はスレッドグループを右クリック。

メニューから、

```
> 追加＞サンプラー＞HTTPリクエスト
```

を選択。  

![](/images/article/jmeter-3.png)

### 「HTTPリクエスト」に以下を設定

![](/images/article/jmeter-4.png)

- サーバー名またはIP：対象のサーバ。
- ポート番号：80、443など。
- Timeouts：空でも良いようだがリクエストをどれだけJMeterで待つか設定。msなので1分なら60000と入力。
- パス：トップページなら不要だが特定のURLの場合はそれを入力。例）example.com/list/なら、「/list/」と入力。
- すべてのイメージとアプレットを繰り返しダウンロードする（HTMLのみ）：オン
- オプションタスク＞モニタとして使用：オン

設定が終わったら、名前をつけて保存、.jmxファイルが生成される。





## 負荷テスト実行サーバに設定ファイルの.jmxをアップロード

実行サーバにアップしておく。実行サーバはUnix系、Linuxが望ましい。
Windows端末でやると、特にスレッドグループを無限ループする際にエラーが頻発したり、Javaがフリーズしたりする。
GUIで設定だけ行い、Linuxサーバに設定ファイルをアップして、実行自体はコマンドラインで行うのが安定稼働してよい。



## 負荷テスト実行サーバにjavaをインストールし、JMeterのtar.gzをダウンロード、解凍

Apache JMeterのtgzをダウンロードし、解凍する。
以下から最新のJMeterの圧縮ファイルのアドレスを確認。

[Index of /dist/jmeter/binaries](http://www.apache.org/dist/jmeter/binaries/)

確認したらwgetする。

```
$ wget http://www.apache.org/dist/jmeter/binaries/apache-jmeter-2.11.tgz
$ tar zxvf apache-jmeter-2.11.tgz
$ cd apache-jmeter-2.11
```

binディレクトリのjmeterが実行ファイル。引数にjmxファイルを与えるとその設定で実行が開始される。

```
$ cd bin
$ ./jmeter -n -t [jmxのファイルパス]
```

`-l ファイル名`を追加するとログをCSV形式で出力してくれる。
`-j ファイル名`とすると、jmeter実行から終了までの詳細なログを出力してくれる。

```
$ ./jmeter -n -t [jmxのファイルパス] -l [ログファイル名]
```

ログファイルの中身（例）
```
1393600309710,209,HTTP リクエスト,200,OK,スレッドグループ 1-378,text,false,302873,112
```


### ログファイル（csv）の形式

出力されたCSVは以下の並びで出力される模様。

```
timeStamp|time|label|responseCode|threadName|dataType|success|failureMessage
```



### コマンドライン引数を変数として使用する

```
$ ./jmeter -n -t sample.jmx -variable=100
```

あらかじめGUIでの入力項目に`${__P(variable)}`と入れておくと、引数`variable`で渡した値`100`がその入力項目に反映される。便利。

参考：[【jmeter】 コマンドライン起動 - たんたんめん日記](http://dnond.hatenablog.com/entry/2013/03/03/202851)
