---
layout: post
title: 'みやすいヘルプコマンドをシェルで書くちょっとしたテクニック'
date: 2019-01-18
comments: true
categories: Linux
published: true
use_toc: true
description: '主にインフラオペレーションの文脈でMakefileを使うことは多いと思いますがそのときのhelp出力の話です。コマンド例を見やすく出力したり色付けしたりするなどのちょっとした小技を紹介します。'
---

自分が使うためのコピペ用ですが、せっかくなので解説もしようかと思います。

コードは以下です。

```Makefile
.DEFAULT_GOAL := help

help: ## print this message
	@echo "Example operations by makefile."
	@echo ""
	@echo "Usage: make SUB_COMMAND argument_name=argument_value"
	@echo ""
	@echo "Command list:"
	@echo ""
	@printf "\033[36m%-30s\033[0m %-50s %s\n" "[Sub command]" "[Description]" "[Example]"
	@grep -E '^[/a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | perl -pe 's%^([/a-zA-Z_-]+):.*?(##)%$$1 $$2%' | awk -F " *?## *?" '{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'
```

perl と awk で頑張っています。
最後のawk `awk -F " *?## *?" '{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'` がミソです。

これを踏まえた make target の書き方は以下のようなかんじです。

```Makefile
all: test build ## run all commands ## make all

build: ## build docker dontainer ## make build argument_name_1=argument_value_1
  @docker build .

test: ## test the node app ## make test NODE_ENV=production
  @npm test $(MAKEFLAGS)
```

` ## ` を区切り文字として、最初の区切りのあとに説明文を、次の区切りのあとにコマンド例を書きます。

そうすると、helpの出力はおよそ以下のようになります。

```sh
$ make
Example operations by makefile.

Usage: make SUB_COMMAND argument_name=argument_value

Command list:

[Sub command]        [Description]                   [Example]
all                  run all commands                make all
build                build docker dontainer          make build argument_name=argument_value
test                 test the node app               make test NODE_ENV=production
```

Sub command, Description, Example の alignment をきれいに揃えられるところが便利で気に入っています。
また Sub command の列は cyan で色付けされてちょっときれいです。

awkでやってるprintfの `'{printf "\033[36m%-30s\033[0m %-50s %s\n", $$1, $$2, $$3}'` あたりの解説ですが、

色付けは `033[36m` 〜 `\033[0m` でおこなっていて、
`%-30s` の `-30` の数値で整列させる幅（スペース埋め）の調整をしています。
埋めないのならば `%s` と書けばよいです（3列目、`$$3`の項）。

地味ですが、忘れかけた頃などふとしたときに便利です。よければ使ってください。
