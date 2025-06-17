---
layout: post
title: "mac と wsl でちゃんと true color に対応する"
date: 2025-06-17 09:00:00 +0900
categories: Tmux
published: true
use_toc: false
---

mac と wsl でちゃんと true color に対応する

neovimで virtual text なりをあれこれ設定していると地味に true color にしたほうが見え方的に都合がいいことが多く、重い腰を上げてちゃんと対応することに。

## 前提

ターミナルアプリは true color に対応しているものを使うこと。

* warp
* wezterm
* alacritty
* iterm2

などなど。

またターミナルマルチプレクサには相変わらず tmux を使っている。手癖になっちゃって乗り換えがだるい。

なお混乱の元なので `$TERM` はzshrc,bashrcなどで自分で set しないように。

## 共通確認コマンド

以下を打ってグラデーションが滑らかに出力されていればOK

```
curl -s https://gist.githubusercontent.com/lifepillar/09a44b8cf0f9397465614e622979107f/raw/24-bit-color.sh | bash
```


## macos

termiinfo
```
brew install ncurses
infocmp tmux-256color > tmux-256color.info
sudo tic -xe tmux-256color tmux-256color.info
rm tmux-256color.info
```

.tmux.conf
```
set-option -g default-terminal "tmux-256color"
set-option -ga terminal-overrides ',screen-256color:Tc'
```

overridesは `*` でもいいのかも。

## wsl(ubuntu)

terminfo
```
sudo apt install ncurses-term -y
```

.tmux.conf
```
set-option -g default-terminal "tmux-256color"
set-option -ga terminal-overrides ",*:Tc"
```

## vim

`set termguicolors` がされてればOK

## 参考

- https://gist.github.com/bbqtd/a4ac060d6f6b9ea6fe3aabe735aa9d95
- https://www.pandanoir.info/entry/2019/11/02/202146
