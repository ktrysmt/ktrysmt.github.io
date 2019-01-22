---
layout: post
title: 'nexe触ってみた'
date: 2019-01-22
comments: true
categories: NodeJS
published: true
use_toc: true
description: 'とりあえず基本的な使い方をなぞってみました。シングルバイナリ吐いてくれるのはインフラおじさん的にはすごくありがたく...今後良く使うことになりそうです。'
---

## インストール

```
npm i -g nexe
```

## 利用するサンプルコード

今回は[公式のexample](https://github.com/nexe/nexe/blob/master/examples/express-app/index.js)を使います。

expressサーバを建ててますね。

```
mkdir /tmp/nexe
cd $!
cat <<'EOF' > app.js
const express = require('express')
const path = require('path')
const app = express()
app.use('/', express.static(path.join(__dirname, 'public')))

app.listen(8888)
EOF
```

## とりあえずMacでビルド

```
nexe app.js
ℹ nexe 2.0.0-rc.34
✔ Compiling result
✔ Entry: 'app.js' written to: app
✔ Finished in 3.066s
```

ターゲットがビルド環境と一緒ならこれだけでOKだそうです。パイプで渡すこともできるみたいです。

なお初回はネイティブコンパイラをDLするのでちょっと時間かかります。

```
ls .
app           app.js        tsconfig.json
```

lsするとtsconfig.jsonも一緒に生成されてますが、これは一旦es5にトランスパイルするために必要みたいです。

あとは

```
./app
```

と実行して、ブラウザやcurlで http://localhost:8888/ へアクセスすれば確認できます。



## ターゲットを変更してビルド

よく使う linux 64bit でやってみます。

```
nexe -t linux-x64 -o app.linux app.js
```

targetオプションで指定できるアーキテクチャのバリエーションは[readme](https://github.com/nexe/nexe#target-string--object)にあります。

ついでに Docker for Mac ですが雑にテストしてみました。

```
docker run -it -v /tmp/nexe:/root -p 8888:8888 ubuntu bash

root@aa623a36cbff:/# /root/app.linux
```

外からcurlします、

```
curl http://localhost:8888
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>Cannot GET /</pre>
</body>
</html>
```

ちゃんと返ってきました。クロスコンパイルも問題なしですね。


私的にここ最近で一番刺さったOSSでした。TypeScriptも盛り上がりを見せていますし、これは色々とはかどりそうです。