---
layout: post
title: "[bash] Gitのコマンドやブランチ名を入力補完"
date: 2014-02-02 15:09:08 +0900 
comments: true
categories: Git
published: true
---


[Git入門 - Gitのコマンド補完](http://www8.atwiki.jp/git_jp/pages/29.html)

上記のサイトに触発されてGit環境改善に乗り出す。

## gitをインストール

以下のサイトを参考に1.8.2をインストール。

[CentOSに最新のgitをインストールする - clavierの日記](http://clavier.hatenablog.com/entry/2013/05/18/204050)

コマンドの記述だけ引用すると以下の通り。

```
# yum install -y zlib-devel perl-devel gettext gcc curl-devel
# wget http://git-core.googlecode.com/files/git-1.8.2.3.tar.gz
# tar xvfz git-1.8.2.3.tar.gz
# cd git-1.8.2.3
# ./configure 
# make
# make install
# git --version
git version 1.8.2.3
```

ところが、私の環境がCentOS6.5だったからなのか？、`make install`に失敗した。
ログを見る限り実行ファイルはあるみたいだったので、シンボリックリンクをはって対応。

```
# ln -s /usr/local/bin/git /usr/bin/
```

## 入力補完シェルの場所を確認する

```
$ locate git-completion.bash
/usr/share/doc/git-1.8.2.1/contrib/completion/git-completion.bash
$ locate git-prompt.sh
/usr/share/doc/git-1.8.2.1/contrib/completion/git-prompt.sh
```

今回は上記に保存されていた。

## .bashrcを編集

```
$ vi $HOME/.bashrc
```

.bashrcに以下の記述を追加。

```
source /usr/share/doc/git-1.8.2.1/contrib/completion/git-prompt.sh
source /usr/share/doc/git-1.8.2.1/contrib/completion/git-completion.bash
GIT_PS1_SHOWDIRTYSTATE=true
export PS1='\[\033[1;32m\]\u@\h\[\033[00m\]:\[\033[1;34m\]\w\[\033[1;31m\]$(__git_ps1)\[\033[00m\][\t]$ '
```

記述を反映。

```
$ source $HOME/.bashrc
```

`$(__git_ps1)`という記述、これはカレントのディレクトリがgitリポジトリである場合、カレントのブランチ名を表示する。
Gitのブランチ名を参照できるようになってるのは、以下のサイトを参考にした。

[ターミナルでgitのコマンドを補完したりブランチ名を表示する – macでgitを便利に使うために – - PPl@ce](http://pplace.jp/2013/12/1601/)

ついでに、好みで`[\t]`も追加している。プロンプトにタイムスタンプが欲しいと思うことが多いので。

## 参考
+ [CentOSに最新のgitをインストールする - clavierの日記](http://clavier.hatenablog.com/entry/2013/05/18/204050)
+ [Git入門 - Gitのコマンド補完](http://www8.atwiki.jp/git_jp/pages/29.html)
+ [ターミナルでgitのコマンドを補完したりブランチ名を表示する – macでgitを便利に使うために – - PPl@ce](http://pplace.jp/2013/12/1601/)
