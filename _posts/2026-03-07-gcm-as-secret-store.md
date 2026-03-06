---
layout: post
title: "Git Credential Managerをシークレットストアとして使う"
date: 2026-03-07 00:01:00 +0900
categories: Shell
published: true
use_toc: false
---

Git for Windowsに同梱されるGit Credential Manager (GCM) を、APIキー等の環境変数管理に流用する話。

## 仕組み

GCMは本来Git認証用だが、`protocol`/`host`/`username`/`password`の組み合わせで任意の値を保存できる。`protocol=custom`、`host=env`などてきとーに名前空間を切って、Git認証と衝突しないようにすればOK。

```
protocol=custom
host=env
username=ANTHROPIC_API_KEY
password=sk-ant-xxxxx
```

あとはGCMの実行ファイルを直接叩いて`store`/`get`/`erase`を呼ぶ。


## 実装

```zsh
_GCM_EXE=$(git config --get credential.helper 2>/dev/null)
if [[ "$_GCM_EXE" == *credential-manager* ]]; then
  GCM_ENV_KEYS=(
    ANTHROPIC_API_KEY
    GEMINI_API_KEY
    EDINETDB_API_KEY
    FALLBACK_OPENROUTER_TOKEN
  )

  _gcm-call() {
    local action=$1; shift
    printf '%s\n' "$@" "" \
      | timeout 1 zsh -c 'eval "$1" "$2"' -- "$_GCM_EXE" "$action" 2>/dev/null
  }

  gcm-get() {
    _gcm-call get "protocol=custom" "host=env" "username=$1" \
      | sed -n 's/^password=//p'
  }

  gcm-set() {
    local key=${1:u} val
    printf 'Enter value for %s: ' "$key"
    read -rs val; echo
    [[ -z "$val" ]] && return 1
    _gcm-call store "protocol=custom" "host=env" "username=$key" "password=$val"
  }

  gcm-rm() {
    _gcm-call erase "protocol=custom" "host=env" "username=${1:u}"
  }

  gcm-ls() {
    local key val
    for key in "${GCM_ENV_KEYS[@]}"; do
      val=$(gcm-get "$key")
      if [[ -n "$val" ]]; then
        printf '%s\t%s\n' "$key" "${val[1,4]}****"
      else
        printf '%s\t(not set)\n' "$key"
      fi
    done
  }

  gcm-env() {
    local key val
    for key in "${GCM_ENV_KEYS[@]}"; do
      val=$(gcm-get "$key")
      [[ -n "$val" ]] && export "$key=$val"
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

バックスラッシュ付きなので`"$_GCM_EXE"`でそのまま実行するとパスが見つからない。ここは仕方ないので`eval`で展開。

### timeout + eval

`timeout`は外部コマンドなのでシェルビルトインの`eval`を直接実行できない。

```zsh
# これはNG (evalバイナリを探そうとしてexit 127)
timeout 1 eval "$_GCM_EXE" get

# zsh -c で囲う
timeout 1 zsh -c 'eval "$1" "$2"' -- "$_GCM_EXE" get
```

### fill を避けてもまだGUIプロンプトの抑制が必要

GCMの`get`は存在しないキーに対してWindows GUIダイアログを出す。WSLからはLinux環境変数が届かないので`GCM_INTERACTIVE=false`は効かない。`timeout 1`で1秒待って打ち切る。存在するキーは即座に返るので影響なし。

## おわり

数が多いと無理だが２～３個ならこれで十分。
ubuntu desktopもOK。
