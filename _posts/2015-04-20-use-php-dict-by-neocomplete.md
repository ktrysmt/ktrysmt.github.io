---
layout: post
title: "neocomplete-php重いのでやめて、辞書ファイルをneocompleteで読み込む方法に切り替えた"
date: 2015-04-19 15:09:08 +0900
comments: true
categories: Vim
published: true
---

<http://ktrysmt.github.io/blog/update-vim-7.4/>の続き。

neocomplete-phpいれるとソースを開いて最初の補完時につっかかるというかもたつくのでストレスたまる、のでやめた。

普通に辞書ファイル作って読み込む方法に切り替え、GoやRuby等他の言語でも同じやり方でできそう。


## Dependencies

- vim 7.4
- neobundle
- neocomplete

## Install

```
wget http://coderepos.org/share/browser/lang/php/misc/dict.php?format=txt -O ~/dict.php
mkdir -p ~/.vim/dictionaries/
php ~/dict.php | sort > ~/.vim/dictionaries/php.dict
```

## Configure .vimrc

neocomplete-phpの設定を除去

```
-NeoBundle 'violetyk/neocomplete-php.vim'
+"NeoBundle 'violetyk/neocomplete-php.vim'
-let g:neocomplete_php_locale = 'ja'
+"let g:neocomplete_php_locale = 'ja'
```

辞書読み込み箇所にPHPを追加

```
" Define dictionary.
let g:neocomplete#sources#dictionary#dictionaries = {
   \ 'default' : '',
   \ 'vimshell' : $HOME.'/.vimshell_hist',
+   \ 'php' : $HOME.'/.vim/dictionaries/php.dict',
   \ 'scheme' : $HOME.'/.gosh_completions'
   \ }
```

## References

- <http://d.hatena.ne.jp/heavenshell/20090115/1232031913>

