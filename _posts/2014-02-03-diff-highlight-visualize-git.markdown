---
layout: post
title: "gitのdiffを見やすくするdiff-highlight"
date: 2014-02-01 15:09:08 +0900
comments: true
categories: Git
published: true
---

[Git の diff を美しく表示するために必要なたった 1 つの設定 #git - 詩と創作・思索のひろば (Poetry, Writing and Contemplation)](http://motemen.hatenablog.com/entry/2013/11/26/Git_%E3%81%AE_diff_%E3%82%92%E7%BE%8E%E3%81%97%E3%81%8F%E8%A1%A8%E7%A4%BA%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AB%E5%BF%85%E8%A6%81%E3%81%AA%E3%81%9F%E3%81%A3%E3%81%9F_1_%E3%81%A4%E3%81%AE%E8%A8%AD)

に触発されたのだが、
いまいちサクッと入れられなかったので、自分の環境で試した時の手順を以下にまとめておく。

Gitのバージョンが古いとおそらくdiff-highlightがGitインストール時に同封されていないので、1.8系へのバージョンアップを試みる。

参考：[Gitのコマンドやブランチ名を入力補完 - Sys Kit](http://kotaroyoshimatsu.github.io/blog/git-input-completion/)


```
$ vi ~/.gitconfig
```

```
[pager]
        log = diff-highlight | less
        show = diff-highlight | less
        diff = diff-highlight | less
```

diff-highlightを探す。

```
$ locate diff-highlight
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight/README
/usr/share/doc/git-1.8.2.1/contrib/diff-highlight/diff-highlight # 見つけた
```

実行できるようにパスを通す

```
$ ln -s /usr/share/doc/git-1.8.2.1/contrib/diff-highlight/diff-highlight /usr/bin/
```

`git diff`してもカラーリングされない、白黒のままだ、っていう場合は`git config`でのカラーリングの設定ができていない。

```
$ git config --global color.ui true
```

