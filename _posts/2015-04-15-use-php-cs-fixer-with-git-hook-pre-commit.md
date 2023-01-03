---
layout: post
title: "git-hookのpre-commitでphp-cs-fixerを叩く"
date: 2015-04-15 15:09:08 +0900
comments: true
categories: PHP
published: true
---

php-cs-fixerはphp5.3以上を要求するので注意。

インストールはcomposer経由でもなんでも。

とりあえずリポジトリのROOTにpharおいてしまうという大雑把なやり口なのであまり真似しないように。

## Install

```
wget http://get.sensiolabs.org/php-cs-fixer.phar
mv php-cs-fixer{.phar,}
chmod +x php-cs-fixer
```

## Edit .git/hooks/pre-commit

```
touch .git/hooks/pre-commit
chmod +x !$
```

で、`.git/hooks/pre-commit`に以下を追加

```
#!/bin/bash

# ディレクトリ定義
ROOT_DIR=$(git rev-parse --show-toplevel)
LIST=$(git status | grep -e '\(modified:\|new file:\)'| grep '\.php' | cut -d':' -f2 )

# php-cs
for file in $LIST
do
  RESULT=$($ROOT_DIR/php-cs-fixer --level=psr2 fix $ROOT_DIR$file --fixers=-psr0)
done
```

## 備考

古いFWだと名前空間という概念がないおかげでクラス名が非常に重要だったりする。
つまりクラス名を勝手に書き換えられるのは困るので`-psr0`としている。
