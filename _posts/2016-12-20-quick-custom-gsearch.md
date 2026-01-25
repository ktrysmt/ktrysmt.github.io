---
layout: post
title: ググったあとワンクリックで期間指定ができるChrome拡張を作った
date: 2016-12-20 09:00:00 +0900
categories: Chrome
published: true
use_toc: false
---

(2017/02/04追記)

- 英語のページのみ検索、ローカル言語（日本語等）のみ検索の選択機能を追加しました。英語記事を探しやすくなります。

(2017/01/07追記)

- 選択肢に新たに「3ヶ月」を追加しました。
- www.google.com, www.google.co.uk などほぼすべての地域のGoogle検索に対応しました。
- 不具合を修正しました。

---

この記事は [Recruit Engineers Advent Calendar 2016](http://www.adventar.org/calendars/1617) の20日目の記事です。

日々の情報収集で役立ちそうなChrome拡張を作ってみたので、今回はそちらの紹介をします。

## これは何
- [Quick Custom GSearch - Chrome ウェブストア](https://chrome.google.com/webstore/detail/quick-custom-gsearch/dcdmfmmmmpjgfaffnaokjpifnihmhaon)

Google検索での検索期間の範囲指定、言語指定をワンクリックに行えるようになるChrome拡張です。

## 使い方

拡張を入れた状態でいつも通りGoogle検索をすると、検索画面の左側の余白部分に小さなUIが出るようになります。こちらに予め設定した期間指定が用意されていますので、これをクリックしていただければ自動的に検索した現在日時から遡る範囲を指定することが出来ます。

![capture.png](https://qiita-image-store.s3.amazonaws.com/0/66424/f32b22de-4f54-a96c-5435-b9ed4d22563a.png)

少し拡大。

![capture_zoom.png](https://qiita-image-store.s3.amazonaws.com/0/66424/ed7d4c95-c733-02f3-a7dd-0964b09d3f7a.png)

選択した期間指定や言語指定は基本的には保持しない（その画面限り）なので、検索するたびに必要であれば都度任意の期間指定を選ぶような使い方なります。

## 何が嬉しいの

### 通常の検索では期間指定等をしづらい

通常のGoogle検索で期間指定を行うには、およそ以下のような手順をとります。

1. まずググる
2. `ツール`を選択
3. `期間指定なし`を選択
4. プルダウンから任意の期間を選択（`１時間以内`/`24時間以内`/`1週間以内`/`1ヶ月以内`/`1年以内`）
5. それ以外の期間範囲で絞り込みたい場合は、`期間を指定`を選択してカレンダーUIから具体的な日付を選ぶ

半年や2年、3年といった範囲で絞るには、地味に手間がかかるかと思います。

### 仕様や情報の更新頻度が高いテーマに

１年以内か、数年単位で大幅に仕様が変わるような技術領域においては、通常、検索結果のうち更新日時などを注視されるかと思います。この拡張を使えば大まかではありますが期間を絞った上で検索結果が表示されるようになるので、情報収集の効率が上がるかもしれません。

### 同じ轍を踏まないように

エンジニアのみなさんにおかれましては、日々トレンドを追いかけ新しい技術に取り組んだりキャッチアップをしているかと思われます。私自身、色々な言語やライブラリ、プラットフォームについて調べるにあたり、情報のアップデートが頻繁にある技術領域においては、ただググっただけではいささか古い記事がヒットしてしまうなどしてイマイチ課題が解決しなかったり、先人が苦労したのと同じ轍を踏んでしまうなど、しばしば困っていました。
そういった経験のある方には、わりと嬉しいツールなんじゃないかなぁと思っています。

### 言語指定でのフィルタリング

期間指定と同様に日本語記事のみ・英語記事のみを検索結果に表示させるには少し手間がかかります。この拡張を使えばワンクリックでアクセスできるようになるので、英語圏でない人でも英語記事のみをブラウジングしやすくなったり、外国語のノイズを除去しやすくなったりします。

## 技術的に工夫したところ

### Mutation Observer

Chromeのアドレスバーから検索する場合の他に、Googleのトップページから検索を行う場合もあるかと思います。Google検索でそういった遷移をたどる場合には、DOM構造の一部だけを再描画させる処理となり、画面全体の再描画はしません。可能であればこれにも対応しておこうと思い、SetInterval…ではなく、MutationObserverを使ってDOM要素の監視し遅滞的に適用する実装を追加して対応しました。

また画像検索や動画検索などほかの検索テーマにおいては期間指定の必要性があまり感じられなかったのと、テキスト検索とは微妙にDOM構造が違うものもあるので、それらには無理に適用させずlocation.search,hash,pathname等をチェックしてUIが出ないように対応しています。

### アーキテクチャ・使用ツール

`package.json`からの抜粋です。

```package.json
{
  "devDependencies": {
    "chai": "^3.5.0",
    "eslint": "^3.13.0",
    "eslint-config-airbnb-base": "^11.0.0",
    "eslint-plugin-import": "^2.2.0",
    "jsdom": "^9.9.1",
    "mocha": "^3.2.0",
    "uglify-js": "git://github.com/mishoo/UglifyJS2#harmony",
    "webpack": "beta"
  }
}
```

動作環境がChromeのみでよいのでES6Syntaxを使いやすいのが非常に有り難いのですが、実は難読化やTreeShakingでお世話になっているUglifyJSが通常はES6Syntaxに対応していません。babelに依存せずPureにES6を使いたかったので、UglifyJSについては`harmony`ブランチを指定して対応しました。うまく動いてくれているようです。

テストについては慣れているmocha+chai+jsdomで書いていました。あとは、地味にwebpack2.2を使っています。eslintについてはairbnbに日々お世話になっています、今回はChrome拡張の開発なのでreact分を抜いたairbnb-baseを使いました。

## 注意事項、不具合など

1. ~~Google検索結果にて特定の画面遷移をたどる場合にのみMutationObserverが発火したりしなかったりすることがあるようで、調査中です。。普通に使う（検索してリンク先にジャンプ）分には問題ないと思います。。~~
2. ~~現状日本語版（www.google.cojp）にしか対応していないので、英語版（www.google.com）の対応もしたいなぁと思っています。~~
3. 不具合など見つかった場合には[Githubでソースを公開](https://github.com/ktrysmt/quick-custom-gsearch)しているので、Issueを上げたりしていただけると嬉しいです。
4. 画像検索や動画検索など他の検索でも期間指定できるほうが良いなどの要望があれば、何かしらご連絡いただけると嬉しいです（画像検索ではUIを置く空きスペースが画面上にほとんどないですが…）。

## Links

- [Quick Custom GSearch - Chrome ウェブストア](https://chrome.google.com/webstore/detail/quick-custom-gsearch/dcdmfmmmmpjgfaffnaokjpifnihmhaon)
- [https://github.com/ktrysmt/quick\-custom\-gsearch](https://github.com/ktrysmt/quick-custom-gsearch)
