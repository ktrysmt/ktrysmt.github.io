---
layout: post
title: ".minttyrcを編集してCygwinのカラースキーマを変更する"
date: 2014-09-27 15:09:08 +0900
comments: true
categories: Windows
published: true
---

`.minttyrc`を編集するといろいろ設定を変えられることを知った。

とはいうもののあまり手を加えたくもないので、とりあえずカラーリングだけ変えてみる。

## .minttyrcを編集

```
$ vim ~/.minttyrc # エディタならなんでもいい

ForegroundColour = 131, 148, 150
BackgroundColour =   0,   0,   0
CursorColour     = 220,  50,  47
Black            =   7,  54,  66
BoldBlack        =   0,  43,  54
Red              = 220,  50,  47
BoldRed          = 203,  75,  22
Green            =   0, 200, 132
BoldGreen        =   0, 200, 132
Yellow           = 204, 204, 102
BoldYellow       = 204, 204, 102
Blue             = 102, 153, 204
BoldBlue         = 102, 153, 204
Magenta          = 111,  54, 130
BoldMagenta      = 108, 113, 196
Cyan             =  42, 161, 152
BoldCyan         = 147, 161, 161
White            = 238, 232, 213
BoldWhite        = 253, 246, 227
```

色みをデフォルトより抑えたカラーリングを公開してくれている人が居たのでありがたく拝借する。

`Magenta`だけ少し色がきつかったのでRedを`-100`している。


## 参考

+ <http://www.rosipov.com/blog/mintty-color-scheme-cygwin/>

thank you so much for your soft-color schema.
