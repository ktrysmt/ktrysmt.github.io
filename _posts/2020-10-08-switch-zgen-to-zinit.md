---
layout: post
title: "zgenをzinitに置き換えたらzshの起動時間が500msから100msに改善した"
date: 2020-10-08 00:00:00 +0900
comments: true
categories: Zsh
published: true
use_toc: false
description: ""
---

## (2022/02/18 追記)

[sheldonに移行しました。](./migrate-zinit-to-sheldon/)

---

zgenであまり困ってはいなかったのですがzinitでもうちょっと速く出来そう、ということなので。

## zgenの場合

[](https://github.com/tarjoilija/zgen){:.card-preview}


### setupのおさらい
```sh
git clone https://github.com/tarjoilija/zgen.git ~/.zgen
```

### zgen時代のzshrc設定例
```sh
: "zgen" && {
  source "${HOME}/.zgen/zgen.zsh"
  if ! zgen saved; then
    echo "Creating a zgen save..."

    zgen oh-my-zsh
    zgen oh-my-zsh plugins/git

    zgen load aws/aws-cli bin/aws_zsh_completer.sh
    zgen load zsh-users/zsh-syntax-highlighting
    zgen load zsh-users/zsh-completions
    zgen load lukechilds/zsh-better-npm-completion
    zgen load docker/cli contrib/completion/zsh/_docker
    zgen load docker/compose contrib/completion/zsh/_docker-compose

    zgen save
  fi
}

# 以下略 / export,aliasなど
```

元のプラグインロード周りは上記の状態でした。
`zgen oh-my-zsh` がクセモノで、oh-my-zsh全体をフルロードしてくれる便利な記述で、起動にやや時間がかかる直接的な原因はこの処理ではなかったのですが、
一方で普段使ってる `設定/キーバインド/エイリアス` がoh-my-zsh内のどのファイルに依存するのかをわかりにくくしてしまっていました。

これを調査するのに少し時間がかかりました。

そして、調査・確認しながら移行した結果が以下です。


## zinitに変える

[](https://github.com/zdharma/zinit){:.card-preview}

### setup
```sh
mkdir ~/.zinit
git clone https://github.com/zdharma/zinit.git ~/.zinit/bin
```

### zinitを使ったzshrcの設定例
```sh
: "zinit" && {
  source ~/.zinit/bin/zinit.zsh

  # https://zdharma.org/zinit/wiki/Example-Minimal-Setup/
  # ここは公式の推奨設定を参考に
  zinit wait lucid light-mode for \
    atinit"zicompinit; zicdreplay" \
      zdharma/fast-syntax-highlighting \
    blockf atpull'zinit creinstall -q .' \
      zsh-users/zsh-completions

  # oh-my-zshのファイルをスニペットとして同期的にバルクロード
  # is-snippet for でまとめて読み込む
  zinit is-snippet for \
      OMZL::completion.zsh \
      OMZL::key-bindings.zsh \
      OMZL::directories.zsh


  # oh-my-zshのファイルをスニペットとして非同期でバルクロード
  zinit wait lucid is-snippet for \
      OMZL::git.zsh \
      OMZP::git

  # .zshでないファイルをcompletionファイルとして認識させつつバルクロード
  zinit wait lucid is-snippet as"completion" for \
      OMZP::docker/_docker \
      OMZP::docker-compose/_docker-compose \
      OMZP::rust/_rust \
      OMZP::cargo/_cargo \
      OMZP::rustup/_rustup

  zinit cdclear -q

  # 同期的に呼ばないとうまく保管されないのでiceは使わない
  zinit snippet https://github.com/aws/aws-cli/blob/v2/bin/aws_zsh_completer.sh
  zinit light lukechilds/zsh-better-npm-completion
}

: "color" && { # promptに色付けする部分だけoh-my-zshから抜き出し
  setopt promptsubst
  autoload -U colors
  colors
}

: "extra up and down key-binding" && { # ctrl+p, ctrl+n の挙動が思い通りでなかったので修正しつつ抜き出し
  autoload -U up-line-or-beginning-search
  autoload -U down-line-or-beginning-search
  zle -N up-line-or-beginning-search
  zle -N down-line-or-beginning-search
  bindkey "^p" up-line-or-beginning-search
  bindkey "^n" down-line-or-beginning-search
}

# 以下略 / export,aliasなど
```

### zinit基本設定をかいつまんで説明

```sh
# 必須
source ~/.zinit/bin/zinit.zsh      # zshrc行頭に入れること

# snippet形式
zinit snippet <URL>                # 読み込みたいzshファイルを直接指定できる
zinit snippet OMZ::path/to/file    # oh-my-zshリポジトリのショートハンド
zinit snippet PZT::path/to/file    # preztoリポジトリのショートハンド
zinit snippet OMZT::gnzh           # oh-my-zsh themeを使う場合のショートハンド
zinit snippet OMZL::git.zsh        # oh-my-zsh lib以下を使う場合のショートハンド
zinit snippet OMZP::git            # oh-my-zsh plugins以下を使う場合のショートハンド

# zinit load/light形式
# zgenやzplugと同じような書き心地で設定できる、最も基本的な呼び方
# load  ... tracking report有り、それでも差はごくわずか
# light ... tracking report無し、通常はこちらで
zinit light lukechilds/zsh-better-npm-completion # githubリポジトリを指定できる
zinit light zsh-users/zsh-completions

# ice
# 直後のlight/loadに任意オプションを適用できる 記述が長くなりがちなときに便利
zinit ice wait'0' lucid
zinit light zsh-users/zsh-completions

# オプションいろいろ
# wait              ... 非同期モード。waitはwait'0'と同義で、非同期実行時の待ち時間を指す。依存関係の定義等もできる
# lucid             ... 非同期読み込み時のmessage出力を抑制
# as"completion"    ... zshでないファイルをzsh completionとして読み込む
# src"path/to/file" ... 対象リポジトリ内の特定のファイルを読み込ませたい場合使う
# light-mode for    ... lightをオプションと同時に設定する書き方で使う
# is-snippet for    ... snippetをオプションと同時に設定する書き方で使う
zinit wait'0' lucid light-mode for zsh-users/zsh-completions
zinit wait lucid is-snippet as"completion" for OMZP::docker/_docker
zinit ice src'bin/aws_zsh_completer.sh'
zinit light aws/aws-cli
```

前述の設定例ではOhMyZshプリインの機能のうち有効にしているのはごく一部ですので、追加でほしいものがある場合はショートハンドを活用して個別にloadするといいと思います。

その他syntaxの解説は[こちらのWiki等を参照](https://zdharma.org/zinit/wiki/)。

## パフォーマンス計測してみる

```sh
# 素の状態
% time zsh -i -c exit
zsh -i -c exit  0.01s user 0.01s system 83% cpu 0.023 total

# zgen
$ time zsh -i -c exit
zsh -i -c exit  0.13s user 0.09s system 39% cpu 0.557 total

# zinit
time zsh -i -c exit
zsh -i -c exit  0.03s user 0.04s system 80% cpu 0.085 total
```

だいぶいいんじゃないかなぁと。

## zinitの使い方
```sh
zinit self-update   # zinitのupdate
zinit update        # snippet, plugin, completionのupdate
zinit cclear        # 使われてない補完設定の掃除
zinit delete <name> # snippet, plugin, completionの削除
zinit times         # snippet, plugin, completionごとの速度計測
```

さほど手間もかからずに移行できてよかった。
