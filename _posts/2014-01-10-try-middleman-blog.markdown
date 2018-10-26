---
layout: post
title: "middleman blogのインストールと足りないgemについて"
date: 2014-01-10 02:48:21 +0000
comments: true
categories: Ruby
published: true
---

## middleman-blogのインストール
rubyが入ってるサーバで、gem install middleman-blog実行。

```
$ gem install middleman-blog
$ middleman init project_name --template=blog
```

## config編集

```
$ cd project_name
$ vim config.yml
```

## Javascript Runtime error
Gemfileの末尾に
gem 'therubyracer'
の1行を追加して、
bundle install
で解決することが多い。

```
$ vim Gemfile
gem 'therubyracer'
```

## pagenate がない

```
undefined local variable or method pagination
```

と出たので、
Gemfileに更に追記。

```
$ vim Gemfile
gem "will_paginate", "~> 3.0.pre2"
```
