---
layout: post
use_toc: false
title: "がんばってJMESPathを覚えてawscliの--queryを使いこなす"
date: 2018-04-30 00:00:00 +0000
comments: true
published: false
description: "書き方にクセがあるのが難点ですが、覚えておくとワンコマンドでだいたい事足りるようになるので、覚えておいて損はないです。"
categories: []
---
JMESPathのリファレンスは以下です。

* http://jmespath.org/tutorial.html
* http://jmespath.org/specification.html

### 基本

* 空ブラケットで子要素をトップルートとして取り出せる
* リスト化したいだけなら `Resources[*].Id`の書き方が一番ラク、複数カラムなら`Resouces[*].[Id,Arn]`とカンマ区切りで。
* 要素の添字(ブラケット)内に`?`を置くことで関数モード的な扱いになる
* 関数はとりあえずcontainsを覚えておけばなんとかなる

？シングルクォーテーション＋バッククォートなら、エスケープいらない？ダブルクオーテーションつかってるのがよくないだけ？

### よい書き方、わるい書き方

ためしにユーザー管理ポリシーのArnを取りに行ってみます。

```sh
$ aws iam list-policies --query "Policies[?contains(Arn,\`:999999999999\`)].{Arn:Arn}"
[
    {
        "Arn": "arn:aws:iam::999999999999:policy/CFnExecutablePolicy"
    },
    {
        "Arn": "arn:aws:iam::999999999999:policy/PackerExecutablePolicy"
    }
]
```

これは半角数字の前にコロンを一個入れているのでpython側でstring判定されています、よってValidですが、型への配慮が雑でよくないですね。


```sh
$ aws iam list-policies --query "Policies[?contains(Arn,\`999999999999\`)].{Arn:Arn}"

'in <string>' requires string as left operand, not int
```

一方これだとpython側で自動的にintにキャストされてしまうのでInvalid、contains関数の第二引数はstringを要求します。


```sh
$ aws iam list-policies --query "Policies[?contains(Arn,to_string(\`999999999999\`))].{Arn:Arn}"
[
    {
        "Arn": "arn:aws:iam::999999999999:policy/CFnExecutablePolicy"
    },
    {
        "Arn": "arn:aws:iam::999999999999:policy/PackerExecutablePolicy"
    }
]
```

そして、これがValidでよい例。JMESPathにはto_stringという関数が提供されています。

したがって、aws管理ポリシーだけをcliで一気に取る方法は以下のようになります。

```sh
aws iam list-policies --query "Policies[?contains(Arn,\`arn:aws:iam::aws\`)].{Arn:Arn}"
```

テキストというか一覧でほしいなら以下。`--output`を使えば良い。

```sh
aws iam list-policies --query "Policies[?contains(Arn,\`arn:aws:iam::aws\`)].{Arn:Arn}" --output text
```

結論、JMESPathをawscliの--queryで使う場合にはzshやpythonの型キャストの気持ちを理解しつつ、`--output`を組み合わせることを意識すればまぁまぁ便利。