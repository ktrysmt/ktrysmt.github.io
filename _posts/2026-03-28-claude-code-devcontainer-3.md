---
layout: post
title: "devcontainerとホスト間のUnixソケットがmacOSでは通らないのでsocatのTCPリレーで通す"
date: 2026-03-28 09:00:00 +0900
categories: [Cloud Infrastructure]
published: true
description: "macOS Docker環境ではdevcontainerとホスト間のUnixドメインソケット通信がVM境界を越えられない。マウント設定・環境変数転送・VirtioFSのソケット制約という3つの問題を特定し、socat TCPリレーで解決した記録"
tags:
  - llm
  - docker
  - tmux
  - devcontainer
  - claude-code
  - sre
  - socat
  - macos
---

[前回](https://ktrysmt.github.io/blog/claude-code-devcontainer-2/)・[前々回](https://ktrysmt.github.io/blog/claude-code-devcontainer/)に続きdevcontainerの話です。


ClaudeCode の hooks 機能で、[エージェントの状態（thinking, notification, done など）を tmux のウィンドウ名やスタイルに反映](https://ktrysmt.github.io/blog/tmux-window-claude-status/)するscriptを自作しています。ホスト上では問題なく動くのですが、macOS の Docker環境で devcontainer 内から ClaudeCode を実行すると hooks が silent no-op になり、tmux ステータスが一切更新されなくなっていました。

macOS の Docker 環境では、bind mount した unixdomainソケットにコンテナ内から接続できないようです。VirtioFSのファイル共有はソケットは見せますが、kernelレベルのソケット通信はVM境界を越えさせないようでした。

```sh
$ ls -la /private/tmp/tmux-501/default
srwxrwx--- 1 ubuntu ubuntu 0 Mar 26 18:39 default   # ファイルは見える

$ tmux display-message -p "#{window_id}"
no server running on /private/tmp/tmux-501/default     # 接続は失敗
```

ファイルとしては見えるのにソケット通信が通りません。WSL(ubuntu)ではこの問題は発生しません（VM境界がなく、直接ソケットアクセスが通るという話です）。

## 解決策

### ホスト側でTCPリレー

unixdomainソケットが VM境界を越えられないということなので、ホスト側で socat で TCP リスナーを立て、hook の実行をリレーしつつホストに委譲します。

コンテナ内で tmux コマンドを実行するのではなく、コンテナからはアクションとペイン ID だけを TCP で送信し、ホスト側の socat がそれを受けて hook スクリプトをホスト上で直接実行します。

ホスト側（devcontainer.jsonのinitializeCommandで起動）:

```bash
SOCK=${TMUX%%,*}
command -v socat >/dev/null 2>&1 && [ -S "$SOCK" ] && {
  pkill -f 'socat.*TCP-LISTEN:2489' 2>/dev/null
  socat TCP-LISTEN:2489,bind=127.0.0.1,reuseaddr,fork \
    SYSTEM:'read action pane; TMUX_PANE=$pane bash ~/.claude/hooks/tmux-window-claude-status.sh $action' &
} || true
```

socat の `SYSTEM` アドレスにより、TCP 接続ごとに hook をホスト上で fork 実行します。`read action pane` で TCP から送られた 1 行を分割し、`TMUX_PANE` を設定してから hook を呼びます。

### hook スクリプト側でフォールバック

コンテナ内の bash `/dev/tcp` 疑似デバイスで TCP を直接送信するため、コンテナ側に socat は不要です。

```bash
command -v tmux &>/dev/null || exit 0

action="$1"
pane="$TMUX_PANE"

# Devcontainer内（＝ソケット到達不可）ならTCPリレーへフォールバック
if ! tmux display-message -p '' 2>/dev/null; then
  [ "$DEVCONTAINER" = "true" ] && \
    { echo "$action $pane" >/dev/tcp/host.docker.internal/2489; } 2>/dev/null
  exit 0
fi
```

## まとめ

| 環境 | 直接ソケット | 動作 |
|------|-------------|------|
| ホスト (macOS/WSL) | 接続可 | 従来通り直接 tmux 操作 |
| devcontainer on WSL | 接続可 | unixdomainソケットをmountし直接使用 |
| devcontainer on macOS | 接続不可 | TCP リレーでホストに委譲 |

## おわり

ファイルは見えるのにソケットが通らないという問題を、ubuntuかmacOSかという問題に間違って当てはめてしまい特定に時間がかかりました。。

socat TCP リレーは力技感がありますが、コンテナ側の変更が最小で済むのはよいです。軽量化を諦めなくて済みます。hookはソケットが繋がるならそのまま使う/繋がらないなら TCP で投げる、というシンプルな分岐だけでいけます。

コードはこちら

[](https://github.com/ktrysmt/dotfiles/tree/master/.devcontainer){:.card-preview}

## 参考

- [Docker Desktop: i-want-to-connect-from-a-container-to-a-service-on-the-host](https://docs.docker.com/desktop/features/networking/#i-want-to-connect-from-a-container-to-a-service-on-the-host)
- [socat(1) - Man page](https://man.archlinux.org/man/socat.1)
- [Claude Codeの\-\-dangerously\-skip\-permissionsをdevcontainerで安全に運用する](https://ktrysmt.github.io/blog/claude-code-devcontainer/)
- [続・ClaudeCode の devcontainer 対応をちゃんとやる](https://ktrysmt.github.io/blog/claude-code-devcontainer-2/)

