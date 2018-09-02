---
layout: post
title: "Rustの開発環境はRustupで"
date: 2017-01-23
comments: true
categories: Rust
published: true
use_toc: true
description: "開発に必要な基本セットがだいたい揃ったっぽいのでまとめました"
---

Rustupのおかげで開発環境がだいぶ整ってきましたね。

## Environment

- Ubuntu

とりあえずUbuntu使っておくのが不具合も少なく無難。macOSでも可。

## Version Manager

- [rustup](https://rustup.rs/)

### Install rustup

```
curl https://sh.rustup.rs -sSf | sh
source ~/.cargo/env
```

`rustc`, `cargo`, `rustup`コマンドがインストールされる。
いまのところは`toolchain`は`nightly`を選んでおくとツールが充実していてよい。

### Confirmation

```
rustup show                # view default channel and current version
rustup default <toolchain> # use which stable, beta, nightly
```

## Package Manager

- Cargo

### Should be installed

- Clippy
- Racer
- Rustfix

### How to Install

```
rustup run nightly cargo install clippy
rustup run nightly cargo install racer
rustup run nightly cargo install --git https://github.com/killercup/rustfix.git
git clone https://github.com/rust-lang/rust -b v1.14.0 ~/.cargo/src
```

## Setup Editor (for Vim)

### Edit `.vimrc`

```
Plug 'neomake/neomake'
Plug 'thinca/vim-quickrun', { 'for': ['rust'] }
Plug 'scrooloose/syntastic', { 'for': ['rust'] }
Plug 'rust-lang/rust.vim', { 'for': ['rust'] }
Plug 'racer-rust/vim-racer', { 'for': ['rust'] }

let g:rustfmt_autosave = 1
augroup NeomakeRustConfig
  autocmd!
  autocmd BufWritePost *.rs Neomake! clippy
augroup END
let g:racer_cmd = '$HOME/.cargo/bin/racer'
let g:rustfmt_command = '$HOME/.cargo/bin/rustfmt'
let $RUST_SRC_PATH = '$HOME/.cargo/src'
```

## References

- <http://qiita.com/hinagishi/items/f43538ce8120e483077e>
