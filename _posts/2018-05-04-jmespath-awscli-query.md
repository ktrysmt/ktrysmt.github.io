---
layout: post
use_toc: false
title: "JMESPathを覚えてawscliを使いこなす"
date: 2018-05-24
comments: true
published: true
description: "書き方に少々クセがあるのが難点ですが、使いこなすとワンコマンドでだいたい事足りるようになるのでCIなどで助かります。awscliを使うひとは覚えておいて損はないです。"
categories: AWS
---

JMESPathのリファレンスは以下です。

* http://jmespath.org/tutorial.html
* http://jmespath.org/specification.html

JMESPathはjson形式の構造に対してXPathチックにアクセスできるクエリ言語です。awscliでは--queryオプションがこれに対応しています。

### 最低限覚えておくとよいこと

* `Resources[]`(空ブラケット)と`Resouces[*]`(アスタリスク)は等価、子要素をトップルートとして取り出せる
* リスト化したいだけなら `Resources[].Id`の書き方が一番ラク、複数カラムなら`Resouces[].[Id,Arn]`とカンマ区切りで指定
* 要素の添字(ブラケット)内に`?`を置くことで関数モード的な扱いになる
* 関数はとりあえずcontainsを覚えておけばなんとかなる
    * 例：
    ```sh
    $ aws iam list-policies --query 'Policies[?contains(Arn,`arn:aws:iam::aws`)].Arn'
    $ aws iam list-policies --query 'Policies[?contains(Arn,to_string(`999999999999`))].Arn'
    ```
* containsの第二引数にはバッククォートを使うので、クエリ全体はシングルクォーテーションで囲うとエスケープの手間が省けてよい
* awscliの`--output`を組み合わせられることを意識するとより一層便利に
* Pipeが使える、`aws iam list-policies --query 'Policies[*].{m:PolicyName} | [0]'`のように書ける

### 気をつけるところ

* containsの第一引数にいれる`@`は検査対象がarray型かstring型でなければならない、object型には使えない
* ほか、containsの第二引数(String型を要求)など、各種関数の引数の型に注意

### 所感

私のユースケースとしては結局扱うのはawscliのレスポンスがほとんどなので、
あまり多くの関数を使いこなすようなことはせず、
最低限containsだけ使えるようになっていれば十分なのかなという評価でした。

型やクォートなどに少し気をつける必要はありますが、コマンドやクエリをスニペット化しておけば低コストに運用できそうです。

ただ正直、filtersで済むのなら、可読性のよいfiltersで済ませたいところです...。

### FYI

* [pecoとzshでかんたんsnippet管理](https://ktrysmt.github.io/blog/snippets-management-by-peco-and-zsh/)
