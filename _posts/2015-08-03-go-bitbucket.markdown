---
layout: post
title: "bitbucketライブラリのgo-bitbucket作った"
date: 2015-08-03 15:09:08 +0900
comments: true
categories: Golang
published: true
---

go-bitbucketというgoで使えるbitbucketライブラリを書きました。

Bitbucket REST API v2.0に対応しています。

- <https://github.com/ktrysmt/go-bitbucket>

## 動機

はじめはBitbucketのAPIをコマンドラインで操作するクライアントが欲しかったのだが、Goのbitbucketライブラリがいまいち使いにくくて自分で作ってしまった。

実直にJsonオブジェクトをただただ返す。

## 特徴

正直ほかの既存のものと比べて優位性は無い…。

- v2.0のエンドポイントをひと通りサポート
- メソッドチェーン風にエンドポイントをたどれる書き方
- 認証はまだBasic認証のみ


## Feature

まだまだ色々足りない気がするがとりあえず以下は随時仕上げる。

- OAuth( APIKEY + SECRET) に対応
- Snippet Endpointに対応（GithubでいうところのGist）
- テストコード足りないので足す

## 収穫

- structとinterfaceの理解
- Godoc便利
- 構造体に関数持ちの構造体を埋め込める、面白い
- ポインタ渡し地獄からの生還

ポインタ渡しを何箇所も書いていると正直何がなんだかわからなくなってくる。

だがそれもきちんとstruct定義して埋め込んであげればいいだけという素直な結論に達した。

Godocはほんとうに便利というかきれいなソースコードを書かなければという気になって精神衛生上大変に良い。良いです。

`type struct` と `type interface`はだいぶ理解できたかな。

goroutineがまだ未踏なのでそろそろgoroutineを使った何かに挑戦してみたい。


