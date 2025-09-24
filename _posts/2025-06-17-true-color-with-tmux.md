---
layout: post
title: "mac や wsl で true color に対応する"
date: 2025-06-17 09:00:00 +0900
categories: Tmux
published: true
use_toc: false
---


neovimで virtual text なりをあれこれ設定していると true color にしたほうが目に優しい・見え方的に都合がいいことが多く重い腰を上げてちゃんと対応することに。

## 前提

ターミナルアプリは true color に対応しているものを使うこと。

* warp
* wezterm
* alacritty
* iterm2

などなど。

ターミナルマルチプレクサには相変わらず `tmux` を使っている。手癖になってしまっていて乗り換えがだるい。

また混乱の元なので `$TERM` はzshrc,bashrcなどで自分で set しないように。

## 共通確認コマンド

以下を打ってグラデーションが滑らかに出力されていればOK

```
curl -s https://gist.githubusercontent.com/ktrysmt/8c32c571e4e2f114834dc63425b9f78e/raw/truecolor.sh | bash
```


## macos

termiinfo
```
brew install ncurses
infocmp xterm-256color > xterm-256color.info
sudo tic -xe xterm-256color xterm-256color.info
rm xterm-256color.info
```

.tmux.conf
```
set-option -g default-terminal "xterm-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
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
