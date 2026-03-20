---
layout: post
title: "Git Credential Managerをシークレットストアとして使う"
date: 2026-03-07 00:01:00 +0900
categories: [Developer Tools]
published: true
use_toc: false
description: "Git Credential Manager(GCM)をAPIキー等のシークレットストアとして流用する方法。WSL環境での注意点やzshでのget/set/erase実装例を紹介します。"
tags:
  - git
  - zsh
  - security
  - windows
---

Git for Windowsに同梱されるGit Credential Manager (GCM) を、APIキー等の管理に流用する話。

## 概要

GCMは本来Git認証用だが、`protocol`/`host`/`username`/`password`の組み合わせで任意の値を保存できる。`protocol=custom`、`host=env`などてきとーに名前空間を切って、Git認証と衝突しないようにすればOK。

```
protocol=custom
host=env
username=GEMINI_API_KEY
password=sk-ant-xxxxx
```

あとはGCMの実行ファイルを直接叩いて`store`/`get`/`erase`を呼ぶ。


## 実装

```zsh
_GCM_RAW=$(git config --global --get credential.helper 2>/dev/null)
if [[ "$_GCM_RAW" == *credential-manager* ]]; then
  _GCM_EXE=(${(Q)${(z)_GCM_RAW}})
  if ! whence "$_GCM_EXE[1]" &>/dev/null; then
    unset _GCM_RAW _GCM_EXE
    return 0 2>/dev/null || true
  fi
  unset _GCM_RAW
  readonly _GCM_EXE
  GCM_ENV_KEYS=(
    GEMINI_API_KEY
    ANTHROPIC_API_KEY
  )

  _gcm-call() {
    local action=$1; shift
    printf '%s\n' "$@" "" | timeout 1 "${_GCM_EXE[@]}" "$action" 2>/dev/null
  }

  _gcm-confirm() {
    local reply
    read -q "reply?$1 [y/N] " || { echo; return 1 }
    echo
  }

  gcm-get() {
    _gcm-confirm "Access credential: $1" || return 1
    _gcm-call get "protocol=custom" "host=env" "username=$1" | sed -n 's/^password=//p'
  }

  gcm-set() {
    local key=${1:u}
    _gcm-confirm "Store credential: $key" || return 1
    local val
    printf 'Enter value for %s: ' "$key"
    read -rs val
    echo
    [[ -z "$val" ]] && return 1
    _gcm-call store "protocol=custom" "host=env" "username=$key" "password=$val"
  }

  gcm-rm() {
    _gcm-confirm "Remove credential: ${1:u}" || return 1
    _gcm-call erase "protocol=custom" "host=env" "username=${1:u}"
  }

  gcm-ls() {
    _gcm-confirm "List all credentials" || return 1
    local key val
    for key in "${GCM_ENV_KEYS[@]}"; do
      val=$(_gcm-call get "protocol=custom" "host=env" "username=$key" | sed -n 's/^password=//p')
      if [[ -n "$val" ]]; then
        printf '%s\t%s\n' "$key" "${val[1,2]}****"
      else
        printf '%s\t(not set)\n' "$key"
      fi
    done
  }

  _gcm-key-completion() { compadd -a GCM_ENV_KEYS }
  compdef _gcm-key-completion gcm-set gcm-get gcm-rm
fi
```

## いろいろ細かいケア

### git credential fill は NG

`git credential fill`はキーが見つからないとき対話プロンプトにフォールバックする。GCMの場合はWindows側にGUIダイアログが出てハングする。GCMのexeを直接叩けば、見つからないときは空を返して終わる。

### credential.helper のパスにバックスラッシュが入る

WSL環境では`git config --get credential.helper`すると以下が返る:

例：
```
/mnt/c/Program\ Files\ \(x86\)/Git\ Credential\ Manager/git-credential-manager.exe
```

バックスラッシュ付きなので文字列のまま実行するとパスが見つからない。zshのパラメータ展開`(Q)${(z)...}`で安全にトークン分割＋アンクォートし、配列として保持する。

```zsh
_GCM_RAW=$(git config --global --get credential.helper 2>/dev/null)
_GCM_EXE=(${(Q)${(z)_GCM_RAW}})
# 配列なのでそのまま実行できる
"${_GCM_EXE[@]}" get
```

### fill を避けてもまだGUIプロンプトの抑制が必要

GCMの`get`は存在しないキーに対してWindows GUIダイアログを出す。WSLからはLinux環境変数が届かないので`GCM_INTERACTIVE=false`は効かない。`timeout 1`で1秒待って打ち切る。存在するキーは即座に返るので影響なし。

### 確認プロンプトの追加

シェルのヒストリ展開やエイリアス誤爆で意図せずクレデンシャルが読み出されるのを防ぐため、`gcm-get`/`gcm-set`/`gcm-rm`/`gcm-ls`に`[y/N]`確認を挟む。

```zsh
_gcm-confirm() {
  local reply
  read -q "reply?$1 [y/N] " || { echo; return 1 }
  echo
}
```

## おわり

数が多いと無理だが２～３個とかちょっとした用途くらいなら。平文よりはマシな感じ。
