---
layout: post
title: "suggest.jsで入力・選択後callback関数を呼ぶ"
date: 2014-09-21 15:09:08 +0900
comments: true
categories: JavaScript
published: true
---



[suggest.js][1] 便利です、使わせていただいています。
suggestで出てきた要素を選択した後に関数を走らせたかったので少し手を入れました。そのときのメモです。
もともとhookBeforeSearchというオプションはあるのですがAfterがなかったので。

 [1]: http://www.enjoyxstudy.com/javascript/suggest/

元ソースはこちら：
[suggest.js - 入力補完ライブラリ][2]

 [2]: http://www.enjoyxstudy.com/javascript/suggest/suggest.js

編集した箇所は以下です。

## １．対象ファイル suggest.js本体

72行目付近にオプションを追加

```javascript
// options
  interval: 500,
  dispMax: 20,
  listTagName: 'div',
  prefix: false,
  ignoreCase: true,
  highlight: false,
  dispAllKey: false,
  classMouseOver: 'over',
  classSelect: 'select',
  hookBeforeSearch: function(){},
  hookAfterSearch: function(){}, // ←これを追加
```

380行目付近、moveEnd関数の最後に追加したオプションの呼び出しを追加

```
moveEnd: function() {

    if (this.input.createTextRange) {
      this.input.focus(); // Opera
      var range = this.input.createTextRange();
      range.move('character', this.input.value.length);
      range.select();
    } else if (this.input.setSelectionRange) {
      this.input.setSelectionRange(this.input.value.length, this.input.value.length);
    }

this.hookAfterSearch();  // ←追加
  },
```

## ２．suggest呼び出し側でオプションを追加

```
var start = function(){
 new Suggest.Local("text", "suggest", list,
    // listのあとの引数にhookAfterSearchを追加↓
    {
      hookAfterSearch: function(text) {
        console.log("hello world");
      }
    }
    // 追加ここまで↑
 );
};
window.addEventListener ?
  window.addEventListener('load', start, false) :
  window.attachEvent('onload', start);
```
