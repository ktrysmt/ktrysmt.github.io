---
layout: post
title: "Apache preforkのチューニングまとめ"
date: 2014-05-26 15:09:08 +0900
comments: true
categories: Apache
published: true
---

## preforkの場合

- apacheの親プロセスが消費するメモリ数とシステムが消費するメモリ数を把握する
- apacheの１子プロセスあたりで消費するメモリ数を把握する
- 物理メモリ数から、apacheの親プロセスが消費するメモリ数＋システムが消費するメモリ数を引く
- 引いた残りを１子プロセスあたりで消費するメモリ数で割る
- 割った数を、MacClientの設定値とする

### ポイント

プロセスの上げ下げにもリソースは消費されるので、

```
StartServers
MinSpareServers
MaxSpareServers
ServerLimit
MaxClients
```

すべてMaxClientsの数値と同値にするのが良い。

### 子プロセスの非共有メモリ消費量算出
こちらのperlスクリプトを使ってます。感謝。

Apacheとかforkしたプロセスのメモリチューニングに関するメモとスクリプト - [http://hirobanex.net/article/2013/10/1381407737](http://hirobanex.net/article/2013/10/1381407737)

ここまで書いておいてなんですがPHP使うならnginxの方がいいと思いはじめてる今日この頃。
