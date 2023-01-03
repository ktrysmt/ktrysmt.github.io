---
layout: post
title: "Wgetで静的サイト生成"
date: 2015-03-01 15:09:08 +0900
comments: true
categories: Linux
published: true
---

html拡張子を付ける場合

```
nohup wget --recursive --no-clobber --page-requisites --html-extension --convert-links --restrict-file-names=windows --domains ドメイン名 --no-parent http://ドメイン名/ > LOG.txt &
```

付けない場合

```
nohup wget --recursive --no-clobber --page-requisites --convert-links --restrict-file-names=windows --domains ドメイン名 --no-parent http://ドメイン名/ > LOG.txt &
```

処理ログ抜いておくのと、Windowsでも参照できるようにしておく。

ドメイン名揃えておくと同一ドメイン内のみのリンク（href）だけ辿ってくれる。

えてしてこの手の処理は時間がかかることが多いのでnohupとバックグラウンド処理を付けておく。
