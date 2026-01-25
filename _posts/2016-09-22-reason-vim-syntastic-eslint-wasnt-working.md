---
layout: post
title: Vim + Syntastic + eslint が動かないのはPATHをzshrcに書いていたのが原因だった
date: 2016-09-22 09:00:00 +0900
categories: Zsh
published: true
use_toc: false
---

## 要するに

以下の環境ならこのトラブルに遭遇する可能性が高い。

- Zshを使っている
- Vimを使っている
- nodejs (or nvm等のnodejsバージョン管理ツール) のPATH解決を.zshrcでやってる

## 原因

Vimは.zshrcに書いているPATHは見ない、.zshenvのを見る。

## 解決策

もしnoodejsやnodejsのバージョン管理ツール用のPATHを.zshrcに書いているなら、
その記述を.zshenvというファイルを作ってそっちに書いてあげればいい。
そうするとVimから見たPATHがzsh側で定義するPATHと一致するため、jslintやeslint、flowtypeなど諸々のコマンドが動くようになる。

## 経緯

### Zsh + Vim + Syntastic

普段Zsh&Vimで開発しており、最近ES6をよく使うようになったのでJavascript系のvim-pluginをいくつか入れてみた。
Syntax Highlightingなどは問題なく使えたのだが、SyntaxCheck用にと入れたeslintがどうもうまく動かない、というかVimが認識しない。
CheckerにはおなじみのSyntasticを使っていたので、ググって出てきたIssueやブログ記事を参考にあれこれやってみるものの、うまくいかず。

ググって以下のページなどを参考にいろいろ試してみたが…。

- [eslint validator not working · Issue \#1110 · scrooloose/syntastic](https://github.com/scrooloose/syntastic/issues/1110)
- [Error in eslint checker · Issue \#1302 · scrooloose/syntastic](https://github.com/scrooloose/syntastic/issues/1302)
- [eslint not working? · Issue \#1347 · scrooloose/syntastic](https://github.com/scrooloose/syntastic/issues/1347)

設定をいじりつつSyntasticInfoで確認しても以下のような有様。

```
Syntastic: active mode enabled
Syntastic info for filetype: javascript
Available checker(s): -
Currently enabled checker(s): -
```

Syntasticの設定とかもいろいろいじくってみたが何も変わらずじまい。
例えば以下のような記述。

```
let s:eslint_path = system('PATH=$(npm bin):$PATH && which eslint')
let b:syntastic_javascript_eslint_exec = substitute(s:eslint_path, '^\n*\s*\(.\{-}\)\n*\s*$', '\1', '')
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
let g:syntastic_javascript_checkers = ['eslint']
```

※参考：[ESlint is available but won't enable · Issue \#1736 · scrooloose/syntastic](https://github.com/scrooloose/syntastic/issues/1736)

`b:syntastic_javascript_eslint_exec`じゃなくて`g:syntastic_javascript_eslint_exe`を使うんだみたいな話もあったのでそれもやったけどダメだった。
環境変数が怪しいなのはわかったのだが、肝心の解決方法にはたどり着けず。

数日過ぎて諦めかけていたところ、再度調べてみたら見落としてたのか.zshenvを使うんだ！という[stackoverflow](http://ja.stackoverflow.com/questions/8586/vim-%E3%81%AE-syntastic%E3%81%8C%E3%81%86%E3%81%BE%E3%81%8F%E5%8B%95%E4%BD%9C%E3%81%97%E3%81%AA%E3%81%84)を発見。
早速やってみたら、なんとまぁあっさり動いた。

GoやPHPなどの言語では、普通はグローバルなPATH（/usr/bin/,/usr/local/bin等）にbinaryが置かれる。vim-go経由でgofmt等がちゃんと動いたのはおそらくこれが理由。
グローバルな位置のPATHならVimも当然汲み取ってくれるだろう。
nodejsについては、私の場合は普段nodebrewなどのバージョン管理ツールを使うようにしているため、PATHの解決はreadmeに従い自分でzshrcにexportを書いていた。
これによりVimからnodejs系ツールのPATH解決ができないということになり、今回のトラブルに引っかかったというわけ。

いやはや勉強になりました。
