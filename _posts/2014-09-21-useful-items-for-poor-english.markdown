---
layout: post
title: "開発に欠かせない英語翻訳ツールの設定"
date: 2014-09-21 15:09:08 +0900
comments: true
categories: Tools
published: true
---

PC壊れたときとかのためにメモ

## chrome拡張

ImTranslatorはすごく便利

+ [ImTranslator: Google Translate - Chrome Web Store](https://chrome.google.com/webstore/detail/imtranslator-google-trans/noaijdpnepcgjemiklgfkcfbkokogabh?hl=en)

## 便利なウェブサイト
以下の2つのサイトを良く使います。関数名やフィールド名を決めるときはcodic.jpのほうが使いやすい。

+ [Google 翻訳](https://translate.google.co.jp/)
+ [codic - デベロッパーのためのネーミング辞書](http://codic.jp/)

## chromeのアドレスバーを細工する
chromeの設定から検索エンジンの管理へ進みます。

```
設定 > 検索 > 検索エンジンの管理
```

設定画面

![english-gt](/images/article/english-gt.png)

設定画面に以下のように入力

検索エンジンを追加 | キーワード | URL
---- | ---- | ----
Google翻訳 | gt | "http://translate.google.co.jp/?source=osdd#auto|auto|%s"
Codic | codic | http://codic.jp/search?q=%s


## ランチャーアプリに細工する

私は長らくlaunchyを愛用しているので今回はこちらに設定してみます。

launchyはWindows用ですがMacならAlfredを使えばいいようです。linuxは知らん

```
Launch オプション > プラグイン > ☑ Weby
```

![english-launchy](/images/article/english-launchy.png)

以下を入力。

```
+-----------------+---------------------------------------------------------+
| Name            | URL                                                     |
+-----------------+---------------------------------------------------------+
| GoogleTranslate | http://translate.google.co.jp/?source=osdd#auto|auto|%1 |
| codic.jp        | http://codic.jp/search?q=%1                             |
+-----------------+---------------------------------------------------------+
```

## Codicには公式プラグインがある
紹介しなかったけどCodicには公式でChrome拡張やFirefox用の検索プロバイダなどがありました。

+ [プラグイン | codic](http://codic.jp/plugins.html)

## 参考

+ <https://translate.google.co.jp/>
+ <http://codic.jp/>
+ Alfredでcodicの検索をする - ton-tech-ton <http://ton-up.net/technote/2014/01/04/alfred-codic-workflow/>
+ <http://www.launchy.net/>
