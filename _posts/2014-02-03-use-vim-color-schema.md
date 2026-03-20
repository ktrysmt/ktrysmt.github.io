---
layout: post
title: "Vimでカラースキーマを使う"
date: 2014-02-05 15:09:08 +0900
comments: true
categories: [Developer Tools]
published: true
description: "Vimにカラースキーマを導入する手順。desert.vimのダウンロードから.vimrcでの設定方法までを解説"
tags:
  - vim
  - editor
---

vim でカラースキーマを使う。

まずはカラースキーマをダウンロード。今回は`desert.vim`を採用。gitをインストールしておいてください。

```
$ mkdir -p ~/.vim/colors/
$ cd ~/.vim/colors/
$ git clone https://github.com/fugalh/desert.vim
```

Githubへ行って直接ファイルを落としても可。
最終的に以下のようになっていればよい。

```
$ ls ~/.vim/colors/desert.vim
/root/.vim/colors/desert.vim
```

次に、$HOME/.vimrcを編集。なければ作成し以下を記述。

```
$ vim ~/.vimrc
```

```
:syntax on
:colorscheme desert
```
