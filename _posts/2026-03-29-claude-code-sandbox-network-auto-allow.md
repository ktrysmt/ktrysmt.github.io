---
layout: post
title: "ClaudeCode /sandboxのnetwork許可をhooksで永続化する"
date: 2026-03-29 16:00:00 +0900
categories: [AI Engineering]
published: true
description: "sandboxのnetwork制限で都度発生するdomain許可promptをPostToolUse hookで自動化。TLSエラーからdomainを抽出しsettings.jsonのallowedDomainsへ自動追記するシェルスクリプトの実装とmkdirベースの排他制御"
tags:
  - claude-code
  - sandbox
  - hooks
  - shell
  - llm
---

ClaudeCode のsandboxはファイルシステムとnetworkの両方を制限してくれます。
network制限は `sandbox.network.allowedDomains` で管理されますが、新しいdomainに遭遇するたびに手動で `settings.json` に追記していくのが面倒でした。かといって `"*"` も憚られます。。

PostToolUse hookを使ってnetworkエラーで失敗したdomainを自動で許可リストに追加する仕組みを入れてみました。

## 背景: sandboxのnetwork制限

ClaudeCode のsandboxを有効にすると、Bash コマンドのnetworkアクセスは `allowedDomains` に列挙されたホストのみに制限されます。
許可リストにないホストへのアクセスは、以下のようなエラーで失敗します。

```
failed to get run: Get "https://api.github.com/repos/foo/bar/actions/runs/123":
tls: failed to verify certificate: x509: OSStatus -26276
```

ClaudeCode はこのエラーを検知すると `dangerouslyDisableSandbox: true` で再実行を試み、ユーザーに許可promptを表示します。
ユーザーが承認すればコマンドは通りますが、この許可は一時的なもので永続化されないようで、次回同じdomainに対して再びpromptが出てしまっていました。

## 解決策: PostToolUse hookによる自動追加

### フロー

1. sandbox内の Bash コマンドがnetworkエラーで失敗します
2. PostToolUse hookがエラーパターンを検知します
3. エラーメッセージ中の URL からdomainを抽出します
4. `settings.json` の `allowedDomains` に追記します
5. 現在のセッションでは ClaudeCode が `dangerouslyDisableSandbox` で再試行し、ユーザーが一時承認します
6. 次回以降のセッションでは、追加済みdomainへのアクセスはpromptなしで通ります

## 実装

### settings.json

sandbox設定とhook登録の2箇所を編集します。

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

`matcher: "Bash"` により `Bash`ツール実行時のみhookが発火します。
`WebFetch` などは通常通り`allow`,`deny`のパーミッション設定でカバーします。

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
LOCKDIR="/tmp/claude-sandbox-auto-allow.lock"
INPUT=$(cat)

exit_code=$(printf '%s' "$INPUT" | jq -r '.tool_response.exitCode // 0')
[[ "$exit_code" != "0" ]] || exit 0

stdout=$(printf '%s' "$INPUT" | jq -r '.tool_response.stdout // empty')
stderr=$(printf '%s' "$INPUT" | jq -r '.tool_response.stderr // empty')

# --- Detection patterns ---
# Categorized by: TLS/SSL, Connection/Proxy, DNS
SSL_PATTERNS='tls: failed to verify certificate'   # Go
SSL_PATTERNS+='|x509:'                              # Go (macOS/Linux)
SSL_PATTERNS+='|SSL certificate problem'            # curl
SSL_PATTERNS+='|certificate verify failed'          # OpenSSL (Ruby, Python, Rust, C)
SSL_PATTERNS+='|SSL_connect returned=1'             # Ruby OpenSSL
SSL_PATTERNS+='|OpenSSL::SSL::SSLError'             # Ruby
SSL_PATTERNS+='|ssl\.SSLError'                      # Python
SSL_PATTERNS+='|UNABLE_TO_VERIFY_LEAF_SIGNATURE'    # Node.js
SSL_PATTERNS+='|SELF_SIGNED_CERT_IN_CHAIN'          # Node.js
SSL_PATTERNS+='|CERT_HAS_EXPIRED'                   # Node.js
SSL_PATTERNS+='|DEPTH_ZERO_SELF_SIGNED_CERT'        # Node.js
SSL_PATTERNS+='|ERR_TLS_CERT_ALTNAME_INVALID'       # Node.js
SSL_PATTERNS+='|InvalidCertificate'                 # Rust rustls
SSL_PATTERNS+='|gnutls.*certificate'                # GnuTLS (C/C++)
SSL_PATTERNS+='|UNABLE_TO_GET_ISSUER_CERT'           # Node.js
SSL_PATTERNS+='|ERR_CERT_AUTHORITY_INVALID'           # Node.js (Chromium)
SSL_PATTERNS+='|CERT_UNTRUSTED'                       # Node.js
SSL_PATTERNS+='|PKIX path building failed'            # Java
SSL_PATTERNS+='|unable to get local issuer certificate' # curl/OpenSSL

CONN_PATTERNS='dial tcp.*(Operation not permitted|connection refused)'  # Go
CONN_PATTERNS+='|proxyconnect tcp:'                 # Go proxy
CONN_PATTERNS+='|ECONNREFUSED'                      # Node.js
CONN_PATTERNS+='|ECONNRESET'                        # Node.js
CONN_PATTERNS+='|ETIMEDOUT'                         # Node.js
CONN_PATTERNS+='|ConnectionRefusedError'            # Python
CONN_PATTERNS+='|ConnectionResetError'              # Python
CONN_PATTERNS+='|ProxyError'                        # Python requests
CONN_PATTERNS+='|Proxy CONNECT aborted'             # curl proxy
CONN_PATTERNS+='|tunnel connection failed'          # curl proxy
CONN_PATTERNS+='|Received HTTP code [0-9]+ from proxy'  # curl proxy
CONN_PATTERNS+='|Failed to connect to'              # curl
CONN_PATTERNS+='|i/o timeout'                        # Go
CONN_PATTERNS+='|Connection timed out'               # curl / POSIX
CONN_PATTERNS+='|Connection reset by peer'           # curl / POSIX
CONN_PATTERNS+='|Network is unreachable'             # Linux
CONN_PATTERNS+='|No route to host'                   # Linux

DNS_PATTERNS='Could not resolve host'               # curl
DNS_PATTERNS+='|ENOTFOUND'                          # Node.js
DNS_PATTERNS+='|no such host'                       # Go
DNS_PATTERNS+='|Name or service not known'           # Linux glibc
DNS_PATTERNS+='|nodename nor servname provided'      # macOS
DNS_PATTERNS+='|Temporary failure in name resolution' # Linux
DNS_PATTERNS+='|getaddrinfo.*failed'                 # C/C++
DNS_PATTERNS+='|dns.*failed to lookup'               # Rust
DNS_PATTERNS+='|NXDOMAIN'                           # DNS response
DNS_PATTERNS+='|SERVFAIL'                           # DNS response
DNS_PATTERNS+='|socket\.gaierror'                     # Python
DNS_PATTERNS+='|UnknownHostException'                 # Java
DNS_PATTERNS+='|unable to resolve host address'       # wget
DNS_PATTERNS+='|SocketError.*getaddrinfo'             # Ruby

if ! printf '%s\n%s' "$stdout" "$stderr" \
  | grep -qE "${SSL_PATTERNS}|${CONN_PATTERNS}|${DNS_PATTERNS}"; then
  exit 0
fi

# --- Domain extraction ---
combined=$(printf '%s\n%s' "$stdout" "$stderr")
# URL: http(s)://domain/...
url_domains=$(printf '%s' "$combined" \
  | grep -oE 'https?://[^/"'"'"'[:space:]]+' \
  | sed 's|https\{0,1\}://||' || true)
# Go: dial tcp: lookup domain:
dial_domains=$(printf '%s' "$combined" \
  | grep -oE 'dial tcp: lookup [^ :]+' \
  | sed 's/^dial tcp: lookup //' || true)
# curl: Could not resolve host: domain
resolve_domains=$(printf '%s' "$combined" \
  | grep -oE 'Could not resolve host: [^ ]+' \
  | sed 's/^Could not resolve host: //' || true)
# Node.js: getaddrinfo ENOTFOUND domain
notfound_domains=$(printf '%s' "$combined" \
  | grep -oE 'ENOTFOUND [^ ]+' \
  | sed 's/^ENOTFOUND //' || true)
# curl: Failed to connect to domain port NNN
failed_domains=$(printf '%s' "$combined" \
  | grep -oE 'Failed to connect to [^ ]+ port' \
  | sed 's/^Failed to connect to //;s/ port$//' || true)
# Go: no such host "domain"
nohost_domains=$(printf '%s' "$combined" \
  | grep -oE 'no such host "[^"]+"' \
  | sed 's/^no such host "//;s/"$//' || true)
# General: connect to domain port NNN failed
connect_domains=$(printf '%s' "$combined" \
  | grep -oE 'connect to [^ ]+ port [0-9]+ failed' \
  | sed 's/^connect to //;s/ port [0-9]* failed$//' || true)
# Java: UnknownHostException: domain
unknownhost_domains=$(printf '%s' "$combined" \
  | grep -oE 'UnknownHostException: [^ ]+' \
  | sed 's/^UnknownHostException: //' || true)
# wget: unable to resolve host address 'domain'
wget_domains=$(printf '%s' "$combined" \
  | grep -oE "unable to resolve host address '[^']+'" \
  | sed "s/^unable to resolve host address '//;s/'$//" || true)
domains=$(printf '%s\n%s\n%s\n%s\n%s\n%s\n%s\n%s\n%s' \
  "$url_domains" "$dial_domains" "$resolve_domains" \
  "$notfound_domains" "$failed_domains" "$nohost_domains" \
  "$connect_domains" "$unknownhost_domains" "$wget_domains" \
  | grep -v '^$' | sort -u || true)
[[ -n "$domains" ]] || exit 0

cleanup_lock() { rmdir "$LOCKDIR" 2>/dev/null; }
trap cleanup_lock EXIT
while ! mkdir "$LOCKDIR" 2>/dev/null; do sleep 0.1; done

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

cleanup_lock
trap - EXIT

exit 0
```

### PostToolUse hookが受け取る JSON

hookは stdin で以下の形式の JSON を受け取ります。

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

スクリプトは `tool_response` のフィールドからエラーパターンとdomainを抽出させます。

## 検知パターン

### TLS/SSL エラー

| パターン | ランタイム |
|----------|-----------|
| `tls: failed to verify certificate` | Go (gh, terraform) |
| `x509:` | Go (macOS: OSStatus / Linux: unknown authority) |
| `SSL certificate problem` | curl |
| `certificate verify failed` | OpenSSL (Ruby, Python, Rust, C) |
| `SSL_connect returned=1` | Ruby OpenSSL |
| `OpenSSL::SSL::SSLError` | Ruby |
| `ssl.SSLError` | Python |
| `UNABLE_TO_VERIFY_LEAF_SIGNATURE` | Node.js |
| `SELF_SIGNED_CERT_IN_CHAIN` | Node.js |
| `CERT_HAS_EXPIRED` | Node.js |
| `DEPTH_ZERO_SELF_SIGNED_CERT` | Node.js |
| `ERR_TLS_CERT_ALTNAME_INVALID` | Node.js |
| `InvalidCertificate` | Rust rustls |
| `gnutls.*certificate` | GnuTLS (C/C++) |
| `UNABLE_TO_GET_ISSUER_CERT` | Node.js |
| `ERR_CERT_AUTHORITY_INVALID` | Node.js (Chromium) |
| `CERT_UNTRUSTED` | Node.js |
| `PKIX path building failed` | Java |
| `unable to get local issuer certificate` | curl/OpenSSL |

### 接続/プロキシ エラー

| パターン | ランタイム |
|----------|-----------|
| `dial tcp.*(Operation not permitted\|connection refused)` | Go |
| `proxyconnect tcp:` | Go proxy |
| `ECONNREFUSED` | Node.js |
| `ECONNRESET` | Node.js |
| `ETIMEDOUT` | Node.js |
| `ConnectionRefusedError` | Python |
| `ConnectionResetError` | Python |
| `ProxyError` | Python requests |
| `Proxy CONNECT aborted` | curl proxy |
| `tunnel connection failed` | curl proxy |
| `Received HTTP code NNN from proxy` | curl proxy |
| `Failed to connect to` | curl |
| `i/o timeout` | Go |
| `Connection timed out` | curl / POSIX |
| `Connection reset by peer` | curl / POSIX |
| `Network is unreachable` | Linux |
| `No route to host` | Linux |

### DNS エラー

| パターン | ランタイム |
|----------|-----------|
| `Could not resolve host` | curl |
| `ENOTFOUND` | Node.js |
| `no such host` | Go |
| `Name or service not known` | Linux glibc |
| `nodename nor servname provided` | macOS |
| `Temporary failure in name resolution` | Linux |
| `getaddrinfo.*failed` | C/C++ |
| `dns.*failed to lookup` | Rust |
| `NXDOMAIN` | DNS応答 |
| `SERVFAIL` | DNS応答 |
| `socket.gaierror` | Python |
| `UnknownHostException` | Java |
| `unable to resolve host address` | wget |
| `SocketError.*getaddrinfo` | Ruby |

### domain抽出元

| パターン | 抽出例 |
|----------|--------|
| `http(s)://domain/...` | `Get "https://api.github.com/..."` → `api.github.com` |
| `dial tcp: lookup domain:` | `dial tcp: lookup api.example.com: ...` → `api.example.com` |
| `Could not resolve host: domain` | `Could not resolve host: registry.npmjs.org` → `registry.npmjs.org` |
| `ENOTFOUND domain` | `getaddrinfo ENOTFOUND api.example.com` → `api.example.com` |
| `Failed to connect to domain port` | `Failed to connect to pypi.org port 443` → `pypi.org` |
| `no such host "domain"` | `no such host "api.example.com"` → `api.example.com` |
| `connect to domain port NNN failed` | `connect to crates.io port 443 failed` → `crates.io` |
| `UnknownHostException: domain` | `UnknownHostException: api.example.com` → `api.example.com` |
| `unable to resolve host address 'domain'` | `unable to resolve host address 'registry.npmjs.org'` → `registry.npmjs.org` |

## 補足: キャッシュ書き込みエラーの対策

networkとは別に、`mise` や `uv` のキャッシュ書き込みもsandbox modeによりブロックされます。
`~/Library/Caches/` がsandboxの書き込み許可リストに含まれていないためです。

```
mise WARN  failed to write cache file:
/Users/<user>/Library/Caches/mise/.../exec_env_xxx.msgpack.z
Operation not permitted (os error 1)
```

これは `settings.json` の `env` セクションで `XDG_CACHE_HOME` をsandbox書き込み可能なパスにリダイレクトすることで解決できます。

```json
{
  "env": {
    "XDG_CACHE_HOME": "/tmp/claude/cache"
  }
}
```

`mise` と `uv` は共に XDG 規約に従ってくれるのでこの設定で両方のキャッシュが `/tmp/claude/cache/` 以下に書き込まれるようになります。
`/tmp/claude/` はsandboxの書き込み許可リストに含まれているようなのでこれでいけます。
OS 再起動等でクリアされますが所詮キャッシュなので実害なしです。

## 注意点

- 追加されたdomainが有効になるのは次回セッションからです。現在のセッションでは一時承認が引き続き求められるようです
- エラーメッセージに URL やホスト名が含まれない場合は抽出できませんでした
- ワイルドカード既存エントリとの照合まではしていません (例: `*.github.com` があっても `api.github.com` が個別に追加されます)。定期的に許可ドメインリストを棚卸しする運用がよいかもしれません。


## おわり

しばらく前からdevcontainerの作り込みをちょこちょこやっていましたが、うまく調整すれば軽い仕事なら/sandboxでも十分かもしれません。devcontainerとagent teamsの相性があまりよくなくて調整に苦慮していたのですが、一定ラインまではこれで賄えるかもしれません。

本気でやるとやはりbpftraceがほしいので、vibe気味に一気に走らせるときは直列にdevcontainer内で走らせ、細かくチェックしながら並列で動くときはhostでsandboxと使い分ける感じで今は考えています。



[](https://github.com/ktrysmt/nemoclaw-safe){:.card-preview}
