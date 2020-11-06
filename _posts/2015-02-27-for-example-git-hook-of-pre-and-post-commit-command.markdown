---
layout: post
title: "よく使う.git/hook"
date: 2015-02-25 15:09:08 +0900
comments: true
categories: Git
published: true
---

ときどき使うのでコピペ用にまとめ。

## .git/hook/pre-commit

```
#!/bin/bash

# ディレクトリ定義
ROOT_DIR=$(git rev-parse --show-toplevel)
LIST=$(git status | grep -e '\(modified:\|new file:\)'| grep '\.php' | cut -d':' -f2 )

# syntaxエラーがあるファイルはコミット禁止
ERRORS_BUFFER=""
for file in $LIST
do
ERRORS=$(php -l $ROOT_DIR$file 2>&1 | grep "Parse error")
if [ "$ERRORS" != "" ]; then
if [ "$ERRORS_BUFFER" != "" ]; then
ERRORS_BUFFER="$ERRORS_BUFFER\n$ERRORS"
else
ERRORS_BUFFER="$ERRORS"
fi
echo "Syntax errors found in file: $file "
exit 1
fi
done

# masterブランチ、testingブランチへはコミット禁止
branch="$(git symbolic-ref HEAD 2>/dev/null)" || \
"$(git describe --contains --all HEAD)"
if [ "${branch##refs/heads/}" = "master" ]; then
echo "Do not commit on the master branch!"
exit 1
fi
if [ "${branch##refs/heads/}" = "testing" ]; then
echo "Do not commit on the testing branch!"
exit 1
fi
```


## .git/hook/post-commit

```
#!/bin/bash

# master、testingを除くremoteのマージ済みブランチを削除
git push origin $(git branch -r --merged | grep origin/ | egrep -v "(testing|master)$" | grep -v "HEAD" | sed s~origin/~:~);

# コミットと同時にpush(up-streamの必要あり)
git --git-dir=.git push origin
```

## 参考

- <http://tm.root-n.com/unix:command_operation:find_php_lint>
- <http://ktrysmt.github.io/blog/setting-git-push-by-hook-post-commit/>
