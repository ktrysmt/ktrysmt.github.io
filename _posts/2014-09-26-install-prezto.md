---
layout: post
title: "oh-my-zshが重いのでpreztoに乗り換える"
date: 2014-09-23 15:09:08 +0900
comments: true
categories: Zsh
published: true
description: "oh-my-zshが重いためCentOSにpreztoをインストールする手順。zshのビルドからpreztoの導入、既存設定の退避とアップデート方法まで解説"
tags:
  - zsh
  - centos
  - shell
---


## install

```
yum -y install ncurses-devel git
wget http://downloads.sourceforge.net/project/zsh/zsh/5.0.5/zsh-5.0.5.tar.gz
tar xvzf zsh-5.0.5.tar.gz
cd zsh-5.0.5
./configure && make && make install
```

## 既にzshやoh-my-zshの設定がある場合は退避

```
cd ~/
mkdir zsh_orig
mv .zlogin .zlogout .zprofile .zshenv .zshrc zsh_orig
```

## zsh起動

```
zsh
git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
chsh -s `which zsh`
```

## Updating

```
cd ~/.zprezto
git pull && git submodule update --init --recursive
```
