---
layout: post
title: "Freeter.ioを使ってみたら結構いい感じだった"
date: 2017-03-02
comments: true
categories: Tools
published: true
use_toc: false
description: "試しに使ってみたところ、集中しやすいタスク処理環境が構築できそうだったので。"
---

## What is this

- <https://freeter.io/>
- タスク管理ツールの類で、Webベースで自分専用のダッシュボードを事由にレイアウト、構築できる。

<img style="width:auto;height:auto;" src="/assets/images/blog/time-management-with-freeter/screenshot.png" />

## Summary

* デフォルトでポモドーロテクニックのタイマーがついてる
* 不自由なブラウザ経由でセッション保持機能が強めのWebpageの表示ができる
* ブラウザとしての機能は必要最小限なので余計なネットサーフィン（死語）が発生しにくい、というかほぼムリ
* TodoListやNoteなど、欲しいWedgetは一通り揃ってる印象
* 自分専用のダッシュボードを事由に構築できる
* 常駐するおかげでグローバルホットキー(Ctrl+Shift+F、変更可)を使ってすぐに最前面に呼び出せる

## Main features

* 基本無料、無料版は３プロジェクト５タブまで
* Win/macOS/Linux対応（たぶんElectlon）
* プログラマ・デザイナ・アナリスト向けのサポートに手厚い
* `Webpage Wedget`が秀逸。各種Webアプリの公式サポートをうたう[WebAppsというページ](https://freeter.io/embedding-web-apps)もあるが、
おそらく事実上ほぼどんなWebページも埋め込み可能。
（ちゃんと動作確認してるよという意味かも）

## 便利そうな使い方など

恐らく最も汎用的で使い勝手の良いレイアウトは、`Text`+`ToDoList`+`Webpage`の３カラムで、かつ`Webpage`にはGoogleカレンダーのPC版URLを設定する。
Googleカレンダーのタスクの使い勝手を保管しつつちょっとしたメモ書きも可能で、しかもそれが１画面ですべて収まるというのがGood。
ほぼ日手帳のようなフリーハンドレイアウトのしやすい紙の手帳のPC版、といった感じに使える。

あらかじめCommandsに実行ファイル（アプリ）を登録しておき、Openerウィジェット設定で任意のディレクトリを設定する際に、登録したCommandsを選べる。
私は普段のメモ書きやブログ記事はvscode(+vscode-journal)で書いており、書きたいときにすぐ起動できて、しかもWorkspaceをいちいち選択する必要がないのが嬉しい。

データファイルの保存場所が任意で選べるので、保存先をDropbox,Git,GoogleDriveなどの管理下に置いておけばバックアップとリストアは問題なく出来る。

グローバルホットキーが便利。常時フルスクリーンで使用し、グローバルホットキーで表示非表示の運用にすると寄り道や迷いがなくなる。

１画面に収まるようなタスクのたぐいであれば画面中央上という見やすい箇所にタイマーがあるのでポモドーロしやすそう。

## ボードレイアウトのサンプル

[公式](https://freeter.io/dashboard-examples)に用途別のダッシュボードサンプルが載っている。

[Web Development](https://freeter.io/dashboard-examples/web-development),
[Project Management](https://freeter.io/dashboard-examples/project-management),
[Writing Blog Post](https://freeter.io/dashboard-examples/writing-blog-post)など色々あって、レイアウトや使い方の参考になる。

公式にはないが、AnalyticsやAdwards、SEOや効果測定系のミニダッシュボードとしても使えそうだと思った。

## 感想

総じて「いかにしてウェブブラウザを使わず（経由せず）に済ませるか」を大事にしているような印象を受けた。
Freeterを使い始めた結果、Chromeの新しいタブがいかに中毒性と依存性の高い魔性のようなボタンUIかということを痛感した。
Chromeが起動していなければそのUIに触れることもできないので、タスク処理中の割り込みや無駄な寄り道が減った。
