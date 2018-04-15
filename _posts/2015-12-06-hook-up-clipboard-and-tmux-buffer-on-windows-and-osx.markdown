---
layout: post
title: "(Windows|OSX)でtmuxのコピーバッファをクリップボードに送る"
date: 2015-12-06 15:09:08 +0900 
comments: true
categories: Tmux
published: true
---

Windows、OSX最近両方使うようになった。  
ターミナル作業がメインな状況だと、tmuxのバッファがクリップボードに送れるマウスレスになり、嬉しい。  
そのやり方を簡単にメモ。

## Windows

以下のソフトウェアをインストールしておく。

- MSYS2
- tmux (msys2起動してpacman経由でインストール→`pacman -S tmux`)

最新のMSYS2を使えば、比較的新しいtmux（今だとver2.2）が入ってくれるはず。  
さすれば`copy-pipe`が使える。  
あとはMSYS2 SHELLを起動して、`~/.tmux.conf`に以下を記述すればOK。  

```
bind-key -t vi-copy Enter copy-pipe "cat > /dev/clipboard"
```

## OSX

これはググれば出てくるが一応書いておくとhomebrew前提で、  
以下のソフトウェアをbrew経由でいれて、

```
brew install tmux
brew install reattach-to-user-namespace
```

次に以下を`~/.tmux.conf`に追記すればOK。

```
bind-key -t vi-copy Enter copy-pipe "reattach-to-user-namespace pbcopy"
```
