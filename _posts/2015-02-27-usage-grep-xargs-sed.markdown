---
layout: post
title: "grep,xargs,sedを使ったワンライナー集"
date: 2015-02-21 15:09:08 +0900
comments: true
categories: Linux
published: true
---

## grepして出てきた文字列を置換

```
grep -lr hogehoge path/to/dir | xargs -I X sed -i "s/hogehoge/fugafuga/g" X
```

## リモートのマージ済みブランチを消す

```
git push origin $(git branch -r --merged | grep origin/ | grep -v master | sed s~origin/~:~)
```

## access_logからIPベースでアクセス数を集計

```
cat /var/log/httpd/access_log | awk '{print $1}' | sort | uniq -c
```
