---
layout: post
title: "Kindle for Mac は homebrewではなく mas (mac app store cli)"
date: 2026-04-24 08:30:00 +0900
categories: [Developer Tools]
published: true
description: "Homebrewのkindle caskはもうない。。mas CLI経由での手順とかまとめる"
tags:
  - macos
  - mac-app-store
  - homebrew
  - cli-tools
---



```sh
$ brew info --cask kindle
Error: Cask 'kindle' is unavailable: No Cask with this name exists.

$ brew search kindle
kindle-comic-converter
kindle-comic-creator
kindle-create
kindle-previewer
send-to-kindle
```

Homebrew-caskのIssue #30244 では、Amazonが `kindleformac.amazon.com` のホスティングを変えた結果ダウンロードURLが壊れたと報告されてて以降復活してない。公式ダウンロードもApp Storeに一本化される流れで、配布経路としてはApp Store一択に。

## mas

[mas-cli/mas](https://github.com/mas-cli/mas) はMac App StoreのCLIで、Homebrewで入れられます。

```sh
brew install mas
```

```sh
$ mas install kindle
Error: No apps found in the App Store for bundle ID kindle

$ mas install "Amazon Kindle"
Error: No apps found in the App Store for bundle ID Amazon Kindle
```

`mas install` の引数はApp Store ID（数値）らしい、`search` でIDを引いてから渡します。

```sh
$ mas search kindle
 302584613  Amazon Kindle                   (7.56)
 545519333  Amazon Prime Video              (10.120.1)
 ...

$ mas install 302584613
Password:
==> Downloading Amazon Kindle (7.56)
==> Downloaded Amazon Kindle (7.56)
==> Installing Amazon Kindle (7.56)
==> Installed Amazon Kindle (7.56) in /Applications/Amazon Kindle.app
```

`/Applications/Amazon Kindle.app` に入ります。

## 事前条件

`mas install` は「App Storeアカウントでサインイン済み、かつそのアカウントでアプリを取得済み」が前提です。macOS High Sierra以降はCLIからのサインインができなくなっているので、最初の一回だけApp Store.appを開いてサインインし、対象アプリを「入手」しておく必要があります。無料アプリなら「入手」ボタンを押すだけで購入履歴に紐付きます。

この前提さえ満たせば、以後の新規マシンセットアップは `mas install 302584613` 一行で復元できます。

## dotfiles/Brewfileへの組み込み

BrewfileはHomebrew以外にmasも宣言できます。

```ruby
# Brewfile
brew "mas"

mas "Amazon Kindle", id: 302584613
mas "Keynote",       id: 409183694
mas "Numbers",       id: 409203825
```


## 参考
- [mas-cli/mas](https://github.com/mas-cli/mas)
- [Homebrew/homebrew-cask Issue #30244 -- broken cask, Kindle for Mac cannot be installed](https://github.com/Homebrew/homebrew-cask/issues/30244)
