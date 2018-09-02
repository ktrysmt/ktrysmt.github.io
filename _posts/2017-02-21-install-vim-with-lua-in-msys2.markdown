---
layout: post
title: "Installing Vim8 +lua in MSYS2"
date: 2017/02/21
comments: true
categories: Vim
published: true
use_toc: true
description: "Msys2のRuntimeは不安定だがそれでもvim+luaが使いたい、です…。"
---

## 前置き

インストール方法自体は[参考サイト](http://qiita.com/tomotanakamura/items/5374e8dc47e710219842)とだいたい同じ。
ただしpython3は公式から[64bitのバイナリ](https://www.python.org/downloads/windows/)をいれるほうがよい。
というのも、昨今いくつかのVimプラギンがPython3を要求するのと、Msys2経由で入るPython3だとなぜかTimeパッケージが見当たらないため。

## Installation

### Dependencies and Lua

```sh
pacman -S base-devel msys2-devel git ruby python2
pacman -S ncurses-devel libcrypt-devel gettext-devel
curl -R -O http://www.lua.org/ftp/lua-5.3.4.tar.gz
tar xvzf lua-5.3.4.tar.gz
cd lua-5.3.4/src
make mingw && cd .. && make install
git clone https://github.com/Alexpux/MSYS2-packages.git
cd MSYS2-packages/vim
```

### Apply the patch

```patch
--- a/vim/PKGBUILD
+++ b/vim/PKGBUILD
@@ -75,13 +75,15 @@ build() {
    # make distclean

    ./configure \
+       --enable-fail-if-missing \
        --prefix=/usr \
        --build=${CHOST} \
        --with-features=huge \
        --with-tlib=ncursesw \
        --enable-cscope \
        --enable-multibyte \
-       --enable-luainterp=dynamic \
+       --enable-luainterp \
+       --with-lua-prefix=/usr/local \
        --enable-perlinterp=dynamic \
        --enable-pythoninterp=dynamic \
        --enable-python3interp=dynamic \
```

### Packaging

```sh
makepkg
pacman -U vim(version).pkg.tar.gz
```

Over.
