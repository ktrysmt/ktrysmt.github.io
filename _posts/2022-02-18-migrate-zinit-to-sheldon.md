---
layout: post
title: "zinitが不安なのでsheldonへ移行したらzshの起動が50msと更に速くなった"
date: 2022-02-18 00:00:00 +0900
comments: true
categories: Zsh
published: true
use_toc: false
description: ""
---

[こちらの記事](https://ktrysmt.github.io/blog/switch-zgen-to-zinit/)で書いたとおり長らくzinitを使ってたんですがあるときリポジトリがなくなってforkを使い事なきを得ていましたが、

[](https://github.com/zdharma-continuum/zinit){:.card-preview}

メンテナンスがやや心配になったのとsheldonが安定してきてよさそうとの話を目にしたので入れ替えてみました。

[](https://github.com/rossmacarthur/sheldon){:.card-preview}

## 手順

作業にあたっては予めzshrcなどのバックアップをとっておくといいと思います。

### 初期設定

```sh
$ brew install sheldon
$ cd ~/
$ mv .zshrc .zshrc.bk
$ sheldon init --shell zsh
$ ${EDITOR} ~/.sheldon/plugins.toml
$ echo 'eval "$(sheldon source)"' > .zshrc
```

### 設定ファイル

[こちらの設定](https://ktrysmt.github.io/blog/switch-zgen-to-zinit/)を一部移植します。

```
shell = "zsh"

apply = ["defer"]

[plugins.zsh-defer]
github = "romkatv/zsh-defer"
apply = ["source"]

[templates]
defer = { value = 'zsh-defer source "{{ file }}"', each = true }

[plugins.compinit]
inline = 'autoload -Uz compinit && zsh-defer compinit'

[plugins.zsh-syntax-highlighting]
github = "zsh-users/zsh-syntax-highlighting"

[plugins.zsh-completions]
github = "zsh-users/zsh-completions"

[plugins.ohmyzsh-lib]
github = "ohmyzsh/ohmyzsh"
dir = "lib"
use = ["{completion,key-bindings,directories}.zsh"]

[plugins.asdf]
github = "asdf-vm/asdf"

[plugins.zsh-better-npm-completion]
github = "lukechilds/zsh-better-npm-completion"

[plugins.aws_zsh_completer]
remote = "https://raw.githubusercontent.com/aws/aws-cli/v2/bin/aws_zsh_completer.sh"

[plugins.dotfiles-defers]
local = "~/dotfiles/zsh"
use = ["{!sync,*}.zsh"]

[plugins.dotfiles-sync]
local = "~/dotfiles/zsh"
use = ["sync.zsh"]
apply = ["source"]
```

### かんたんに解説

上記のとおりtomlで設定します。設定ファイルは上から順番に読み込まれるので順番には少し注意する必要があります。
.zshrc内は `eval "$(sheldon source)"` 一行だけ書いて.zshrcには何も設定を置かないことを推奨です。
仮に書いていても挙動を上書きされるのでbindkeyなど一度発火したら以降発火しなくなります。

[基本の設定は公式を読めばわかりますが](https://sheldon.cli.rs/Examples.html)、公式docsに書いてない記述もちらほらもあるので以下で補足します。

* `apply = ["defer"]` これを頭でやっとくとデフォルトの挙動としてdefer（非同期読み込み）をしてくれます。
* `apply = ["source"]` 逆に同期的読み込みを明示するときはこれを追加して対応します。
* `inline = autoload -Uz compinit && zsh-defer compinit` compinitに依存する記述をしている場合はこれを入れておくと期待した動作に

### 自前の設定の組み入れ方

dotfiles管理下の自分の設定をどうするかですが、これまでzshrc一枚で全部書いていたのを今回 `dotfiles/zsh/` 以下に分割して管理することにしました。

```
$ exa -T dotfiles/zsh
dotfiles/zsh
├── alias.zsh
├── bindkey.zsh
├── env.zsh
├── lazyload-completions.zsh
├── ostype.zsh
├── peco-snippet.zsh
├── powered_cd.zsh
├── private-source.zsh
├── setopt.zsh
├── sheldon.plugins.toml
├── sync.zsh
├── tmux-refresh.zsh
└── zprof.zsh
```

個々の設定ですが同期的読み込みでないと期待通りに動いてくれないものに限り `dotfiles/zsh/sync.zsh` にまとめて記述し、それ以外は全て非同期で読み込む…というようにして棲み分けてます。`use = ["{!sync,*}.zsh"]` と書けばsync以外をloadする、という意味です。直感的でいいですね。`sync.zsh` のサイズが目立ってきたら同期的読み込みのほうもファイル分割するかもしれません。

`shelodn.plugins.toml` は冒頭のtomlのことで、端末セットアップ時には

```
mkdir ~/.sheldon/ && ln -s ~/dotfiles/zsh/sheldon.plugins.toml ~/.sheldon/plugins.toml
```

を実行します。
なお.zshenvは利用継続してます（一部の用途でzshenvでないと都合が悪くなるため）。

## 更新は

`sheldon lock --update` やっとけばOKです。本体は `brew upgrade` で。

## 小ネタ

`defer` を使ってbindkeyなどを呼ぶ場合はなにかひとつキーを押さないとloadされないので注意です。terminal起動直後にすぐ設定したキーバインドを使いたい場合は、目的のbindkeyだけはdeferを使わず同期的に読み込んであげる必要があります。一部のsetoptも同様です。

このへんは触りながら微調整するといいと思います。

## 計測

```
# zinit
time zsh -i -c exit
zsh -i -c exit  0.03s user 0.04s system 80% cpu 0.085 total

# sheldon
$ time zsh -i -c exit
zsh -i -c exit  0.02s user 0.02s system 76% cpu 0.055 total
```

多少ブレはありますがzinitでは平均80ms前後だったのがsheldonにすることで平均50ms前後に。20-30msくらい改善しました。
updateも速いし設定もtoml形式で直感的になりzshrcも整理されたので、私自身は移行してよかったです。
ただ、速度を求めるだけならzinitと大差ないので無理に移行する必要はないかも。

