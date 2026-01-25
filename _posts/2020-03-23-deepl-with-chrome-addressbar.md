---
layout: post
title: DeepL翻訳をchromeのアドレスバーからすぐ呼べるようにする
date: 2020-03-23
categories: Chrome
published: true
use_toc: false
---

久々の投稿なので、リハビリ代わりに軽いものを。

といってもやってる人多い、既出ネタだとは思いますが。

## 設定

1. <chrome://settings/searchEngines> にアクセス

2. `検索エンジン` の `追加` を選択

3. 以下を入力し保存

| name              |       value                                   |
|:----------------- |:------------------                            |
| 検索エンジン      | DeepL                                         |
| キーワード        | dl                                            |
| URL（%s=検索語句）| https://www.deepl.com/translator#auto/auto/%s |



## 使い方

1. アドレスバーで `dl` と入力
2. TABキーを押す
3. 翻訳にかけたい文章を入力
4. Enter

## 補足

* `#auto/auto/` の部分は `翻訳元言語/翻訳先言語` に相当するので `#en/ja` とすれば 英→和 に固定。
* キーワード(`dl`)の部分は自分にとって打ちやすい任意の文字列に変えるといいと思います。
* もともとは同じことを長らく Google Translate でやってました。それを流用しただけです。

※Google Translateの場合の設定例

| name              |       value                                  |
|:----------------- |:------------------                           |
| 検索エンジン      | Google Translate                |
| キーワード        | gt                                |
| URL（%s=検索語句）| https://translate.google.co.jp/#auto/auto/%s |

