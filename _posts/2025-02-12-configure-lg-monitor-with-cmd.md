---
layout: post
title: "LG OnScreen Controlをcliで操作する"
date: 2025-02-12 23:45:00 +0900
categories: Windows
published: true
use_toc: false
description: "MS-DOSすぐ忘れそうなので備忘。"
---

ヘルプファイルがおよそ以下にある（と思われる）のでこれを参照する。パスが違う場合はeverythingなどで検索。

* `file:///C:/Program%20Files%20(x86)/LG%20Electronics/OnScreen%20Control/bin/Help/GameUICommandLineInterface.html`

ここで`-t`で指定しているDisplay IDは再起動などいろんなタイミングで変わるので、変数として切り出して使う。

例）明るさを50%に設定
```
cd "C:\Program Files (x86)\LG Electronics\OnScreen Control\bin"
for /f "delims=" %%a in ('.\OSCCLI.exe -c list ^| findstr /v "Monitor Display ID"') do set "var=%%a"
".\OSCCLI.exe" -c brightness -t %var:~0,2% -o set 50
```

あとはbatにしてタスクスケジューラで呼び出すなりランチャから呼び出すなり。

cmd.exeなら変数は`%a`だがbatの場合は`%%a`であるという仕様にハマりかけたがgeminiが助けてくれた。LLMすばらしい。
