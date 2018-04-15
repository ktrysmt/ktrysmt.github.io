---
layout: post
title: "Goで依存パッケージをとりあえずまとめてインストールしちまいたい"
date: 2016-03-14 15:09:08 +0900 
comments: true
categories: Golang
published: true
---

依存管理する程でもない（めんどくさい）とか、とりあえず入ればいいよみたいな場合はこのワンライナー。

`ghq`があるなら`ghq`経由のほうがすぐ終わる。

```bash
go build 2>&1 | grep cannot | awk -F'"' '{print $2}' | xargs -I _ ghq get https://_
```

`go get`なら以下。golang.orgなど公式系は`ghq`だとError吐くので、それらも入れるなら`go get`で入れる。

```
go build 2>&1 | grep cannot | awk -F'"' '{print $2}' | xargs -I _ go get _"
```

テキトーにつき取り扱い注意。
