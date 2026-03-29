---
layout: post
title: "ClaudeCode /sandboxのnetwork許可をhooksで永続化する"
date: 2026-03-29 16:00:00 +0900
categories: [AI Engineering]
published: true
description: "sandboxのnetwork制限で都度発生するdomain許可promptをPostToolUse hookで自動化。TLSエラーからdomainを抽出しsettings.jsonのallowedDomainsへ自動追記するシェルスクリプトの実装とflockによる排他制御"
tags:
  - claude-code
  - sandbox
  - hooks
  - shell
  - llm
---

ClaudeCode のsandboxはファイルシステムとnetworkの両方を制限してくれる。
network制限は `sandbox.network.allowedDomains` で管理されるが、新しいdomainに遭遇するたびに手動で `settings.json` に追記していくのが面倒だった。かといって `"*"` も憚られる。。

PostToolUse hookを使ってnetworkエラーで失敗したdomainを自動で許可リストに追加する仕組みを入れてみた。

## 背景: sandboxのnetwork制限

ClaudeCode のsandboxを有効にすると、Bash コマンドのnetworkアクセスは `allowedDomains` に列挙されたホストのみに制限される。
許可リストにないホストへのアクセスは、以下のようなエラーで失敗する。

```
failed to get run: Get "https://api.github.com/repos/foo/bar/actions/runs/123":
tls: failed to verify certificate: x509: OSStatus -26276
```

ClaudeCode はこのエラーを検知すると `dangerouslyDisableSandbox: true` で再実行を試み、ユーザーに許可promptを表示する。
ユーザーが承認すればコマンドは通るが、この許可は一時的なもので永続化されないらしく、次回同じdomainに対して再びpromptが出てしまっていた。

## 解決策: PostToolUse hookによる自動追加

### フロー

1. sandbox内の Bash コマンドがnetworkエラーで失敗する
2. PostToolUse hookがエラーパターンを検知する
3. エラーメッセージ中の URL からdomainを抽出する
4. `settings.json` の `allowedDomains` に追記する
5. 現在のセッションでは ClaudeCode が `dangerouslyDisableSandbox` で再試行し、ユーザーが一時承認する
6. 次回以降のセッションでは、追加済みdomainへのアクセスはpromptなしで通る

## 実装

### settings.json

sandbox設定とhook登録の2箇所を編集する。

sandbox設定 (`sandbox` セクション):

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "network": {
      "allowedDomains": [
        "mise.jdx.dev",
        "github.com",
        "api.github.com",
        "*.github.com",
        "*.githubusercontent.com"
      ]
    }
  }
}
```

hook登録 (`hooks.PostToolUse` セクション):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/sandbox-auto-allow.sh"
          }
        ]
      }
    ]
  }
}
```

`matcher: "Bash"` により `Bash`ツール実行時のみhookが発火する。
`WebFetch` などは通常通り`allow`,`deny`のパーミッション設定でカバー。

### hookスクリプト

`~/.claude/hooks/sandbox-auto-allow.sh`:

```bash
#!/bin/bash
# PostToolUse hook (matcher: Bash)
# Detect sandbox network errors, extract the blocked domain,
# and append it to sandbox.network.allowedDomains in settings.json.
# Takes effect from the next session.

set -euo pipefail

SETTINGS="$HOME/.claude/settings.json"
LOCKFILE="/tmp/claude-sandbox-auto-allow.lock"
INPUT=$(cat)

exit_code=$(printf '%s' "$INPUT" | jq -r '.tool_response.exitCode // 0')
[[ "$exit_code" != "0" ]] || exit 0

stdout=$(printf '%s' "$INPUT" | jq -r '.tool_response.stdout // empty')
stderr=$(printf '%s' "$INPUT" | jq -r '.tool_response.stderr // empty')

# Detect sandbox-specific network errors only
if ! printf '%s\n%s' "$stdout" "$stderr" \
  | grep -qE 'tls: failed to verify certificate|x509:.*OSStatus|dial tcp.*Operation not permitted'; then
  exit 0
fi

# Extract domains from URLs  e.g. Get "https://api.github.com/..."
# Also handle: dial tcp: lookup api.example.com: ...
combined=$(printf '%s\n%s' "$stdout" "$stderr")
domains=$(printf '%s' "$combined" \
  | grep -oE 'https?://[^/"'"'"'[:space:]]+' \
  | sed 's|https\{0,1\}://||' || true)
dial_domains=$(printf '%s' "$combined" \
  | grep -oE 'dial tcp: lookup [^ :]+' \
  | sed 's/^dial tcp: lookup //' || true)
domains=$(printf '%s\n%s' "$domains" "$dial_domains" \
  | grep -v '^$' | sort -u || true)
[[ -n "$domains" ]] || exit 0

exec 9>"$LOCKFILE"
flock 9

for domain in $domains; do
  if jq -e --arg d "$domain" \
    '.sandbox.network.allowedDomains // [] | index($d)' \
    "$SETTINGS" >/dev/null 2>&1; then
    continue
  fi

  tmp="${SETTINGS}.tmp.$$"
  if jq --arg d "$domain" '
    .sandbox.network //= {} |
    .sandbox.network.allowedDomains //= [] |
    .sandbox.network.allowedDomains += [$d]
  ' "$SETTINGS" > "$tmp" && mv "$tmp" "$SETTINGS"; then
    echo "sandbox-auto-allow: added $domain" >&2
  else
    rm -f "$tmp"
  fi
done

exec 9>&-

exit 0
```

### PostToolUse hookが受け取る JSON

hookは stdin で以下の形式の JSON を受け取る。

```json
{
  "session_id": "abc123",
  "hook_event_name": "PostToolUse",
  "tool_name": "Bash",
  "tool_input": {
    "command": "gh run view 123 --repo foo/bar"
  },
  "tool_response": {
    "stdout": "failed to get run: Get \"https://api.github.com/...\": tls: ...",
    "stderr": "",
    "exitCode": 1,
    "success": false
  }
}
```

スクリプトは `tool_response` のフィールドからエラーパターンとdomainを抽出させる。

## 検知パターン

| パターン | 発生状況 |
|----------|----------|
| `tls: failed to verify certificate` | sandboxプロキシが TLS ハンドシェイクをブロック |
| `x509:.*OSStatus` | macOS の証明書検証がsandboxにより失敗 |
| `dial tcp.*Operation not permitted` | TCP 接続自体がsandboxにより拒否 |

### domain抽出元

- URL パターン: `Get "https://api.github.com/..."` から `api.github.com` を抽出
- DNS パターン: `dial tcp: lookup api.example.com: ...` から `api.example.com` を抽出

## 補足: キャッシュ書き込みエラーの対策

networkとは別に、`mise` や `uv` のキャッシュ書き込みもsandbox modeによりブロックされる。
`~/Library/Caches/` がsandboxの書き込み許可リストに含まれていないため。

```
mise WARN  failed to write cache file:
/Users/dew/Library/Caches/mise/.../exec_env_xxx.msgpack.z
Operation not permitted (os error 1)
```

これは `settings.json` の `env` セクションで `XDG_CACHE_HOME` をsandbox書き込み可能なパスにリダイレクトすることで解決できる。

```json
{
  "env": {
    "XDG_CACHE_HOME": "/tmp/claude/cache"
  }
}
```

`mise` と `uv` は共に XDG 規約に従ってくれるのでこの設定で両方のキャッシュが `/tmp/claude/cache/` 以下に書き込まれるようになる。
`/tmp/claude/` はsandboxの書き込み許可リストに含まれているようなのでこれでいける。
OS 再起動等でクリアされるが所詮キャッシュなので実害なし。

## 注意点

- 追加されたdomainが有効になるのは次回セッションから。現在のセッションでは一時承認が引き続き求められるっぽい
- エラーメッセージに URL やホスト名が含まれない場合は抽出できなかった
- ワイルドカード既存エントリとの照合まではしてない (例: `*.github.com` があっても `api.github.com` が個別に追加される)。定期的に許可ドメインリストを棚卸しする運用がいいかも。


## おわり

devcontainerの作り込みをちょこちょこやっていたが、sandboxをうまく調整すれば軽い仕事ならこれでも十分かもしれない。devcontainerとagent teamsの相性があまりよくなくて調整に苦慮していたのだが、これで賄えるかも。
