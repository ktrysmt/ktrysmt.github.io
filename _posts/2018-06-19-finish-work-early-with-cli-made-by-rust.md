---
layout: post
title: "Rust製ツールを使って定時で帰ろう (ripgrep,fd,exa)"
date: 2018-06-19 23:00:00 +0900
comments: true
categories: Rust
published: true
use_toc: false
description: "各種linuxコマンドをリッチで処理速度も早いRust製のCLIに置き換ると幸せになれるかもという話。タイトルが煽り気味で恐縮ですが、作業効率はわりかし上がると思うのでおすすめです。"
---

今回紹介するのは以下の３つのCLIです。

* ripgrep
* fd
* exa

## ripgrep

> <https://github.com/BurntSushi/ripgrep>

コマンド名は`rg`。`grep`の拡張版。`ag`より若干速い。

* brewで入ります。 `brew install ripgrep`
* `-z` で圧縮ファイルの中身も検索
* `.gitignore` に記載のディレクトリ・ファイルを無視してくれる、すべて含むことも可能
* SJISやEUC-JPなどもサポート、 `-E` で任意のエンコーディング指定が可能
* `-t` でファイルタイプ指定のinclude、 `-T` でファイルタイプ指定のexclude。`-tpy`でPython, `-Tjs`でJSを除外、といった風に。
* `-g` ファイルリストに対してglobできる。`-g '!*.min.js'`といった感じに`!`を入れるとexcludeに反転する。
* `-r` で置換。まず検索して検索結果の末尾に`-r`を追加する流れ。
  * 構文例: `rg 'fast\s+(\w+)' README.md -r 'fast-$1'`
  * ただしripgrepは方針としてoverwriteはサポートしないので注意。置換結果の表示のみ。

globとファイルの中身への検索を組み合わせて`rg foo -g '*.min.js' ./public`とか書けるのがいい。
部分一致でなくパターンマッチングも使えます、`rg -e '^foo' -g '*.md'` こんな感じ。
巨大なリポジトリで探しものをしているときとかに重宝します。

## fd

> <https://github.com/sharkdp/fd>

コマンド名は`fd`。`find`の拡張版。

* brewで入ります。 `brew install fd`
* オプション無しでregexパターンを渡します。`cd /etc && fd '^x.*rc$'`
* 末尾でのディレクトリ指定が直感的でいい。 `fd '^x.*rc$' /etc`
* `-e` 拡張子指定。あんま使わないかも、`fd 'png$'`みたいにパターンマッチを書くことのほうが多い。
* `-d` maxDepthのこと。あんま使わないかな。
* `-E` excludeパターンマッチを書ける。`-i`よりこっちのほうがよく使うかも。`node_modules`の除外などに。
* `-0` 検索結果を`NULL`でセパレートしてくれる、検索結果をリダイレクトしてxargsへ渡したいときに助かります。

総じて、findを使い慣れていればあまり困らないものではありますが、findより直感的に書け、また処理速度がかなり速いので作業効率は上がりやすいです。

## exa

> <https://github.com/ogham/exa>

コマンド名は`exa`。`ls`の拡張版でツリー表示の機能も内包されている。

* brewで入ります。 `brew install exa`
* `-l` 通常のlsっぽい表示になる。デフォルトはgrid表示なのだけど、個人的にちょっと見づらいのでこっちを好んで使います。
* `-T` lsを再帰的に行いかつツリー表示してくれる、これが便利
* `-I` ignoreパターンを書ける、ツリー表示時に`-I "node_modules`などと指定します
* `-L` いわゆるmaxDepth、intで指定。思いがけず階層が深かったときに。

カラフルで見やすい。`exa -lha`, `exa -Tl`あたりをsnippetやaliasに入れておくといいです。私は`alias l="exa -lha"`をzshに入れてます。

## おまけ：Vimとripgrepの連携

vimgrepをrgに置き換えるのもおすすめです。`--sort-files`を付けると速度は少し落ちますが、sort済み結果を返してくれます、`Qfreplace`との組み合わせでよく使います。

```vim
if executable("rg")
    set grepprg=rg\ --vimgrep\ --no-heading
    set grepformat=%f:%l:%c:%m,%f:%l:%m
endif
command! -nargs=* -complete=file Rg :tabnew | :silent grep --sort-files <args>
command! -nargs=* -complete=file Rgg :tabnew | :silent grep <args>
```

fzf連携も便利です。

```vim
command! -bang -nargs=* Ripgrep
  \ call fzf#vim#grep(
  \   'rg --column --line-number --no-heading --color=always '.shellescape(<q-args>), 1,
  \   <bang>0 ? fzf#vim#with_preview('up:60%')
  \           : fzf#vim#with_preview({'options': '--exact --reverse --delimiter : --nth 3..'}, 'right:50%:hidden', '?'),
  \   <bang>0)
```

## 所感

WindowsなどでSJISも対象にしたい場合には無理に`rg`を使わず、`pt`を使うのもおすすめです。Go製なのでバイナリを置くだけですぐ使えますし、速度も`ag`並です。

よく使うツールの手入れといいますか、速くできるところは極力高速化していきたいものです。まぁ、zshrcやvimrcの手入れなんかも含めるとキリがないのですが...。特にripgrepとfdは処理速度の速さが売りなので、よければ入れてみてください。

## 参考

* <https://github.com/BurntSushi/ripgrep>
* <https://github.com/sharkdp/fd>
* <https://github.com/ogham/exa>
* <https://qiita.com/ktrysmt/items/70fa1d4e88e0d362c410>
* <https://github.com/monochromegane/the_platinum_searcher>
* <http://someneat.hatenablog.jp/entry/2017/03/12/011335>
* <https://github.com/thinca/vim-qfreplace>



