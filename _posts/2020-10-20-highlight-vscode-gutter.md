---
layout: post
title: "vscodeで現在の行の行番号だけをhighlightする方法"
date: 2020-10-20 00:00:00 +0900
comments: true
categories: "VSCode"
published: true
use_toc: false
description: ""
---

小ネタです。

デフォルトテーマが好みで使っているのですが、検索やアウトラインで移動したときに今どこかわかりにくいのでこれで対策。

```json
  "settings": {
    :
    "editor.renderLineHighlight": "gutter",
    "workbench.colorCustomizations": {
      "editorLineNumber.activeForeground": "#ffffff",
    }
  }
```

エディタの行自体をハイライトしたりアンダーライン付けたりするとすると画面がうるさくなってうっとおしいので、gutterだけ目立たせたかったのです。
