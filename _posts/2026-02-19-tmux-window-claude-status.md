---
layout: post
title: "ClaudeCodeの進捗状況をtmuxウィンドウ名に反映する"
date: 2026-02-19 23:00:00 +0900
categories: [AI Engineering]
published: true
description: "Claude Codeの思考中・応答完了・権限確認待ちなどの状態をtmuxウィンドウ名と色で可視化するhookスクリプトの実装方法を解説します。"
tags:
  - tmux
  - llm
  - bash
---


ClaudeCode を tmux で使っていると別ウィンドウに切り替えたとき「今あっちの Claude はどうなってるかわかりにくい」問題。これを解消したい。


## 挙動

| 状態 | ウィンドウ名 | 色 |
|---|---|---|
| Claudeセッション開始 | `[--]name` | 緑・太字 |
| Claude考え中 | `[**]name` | 青・太字 |
| Claude応答完了 | `[==]name` | 緑・太字 |
| 権限確認待ち | `[??]name` | 黄・太字 |
| Claudeセッション終了 | `name` (復元) | デフォルト |

prefix `[--]` `[**]` `[==]` `[??]` をつけるだけだと視認性がイマイチなので色を付けるのも同時にやるようにしているが、色かprefixどっちかでもいいかも。

同じウィンドウ内に複数ペインで Claude を起動している場合は、各ペインのシンボルを `|` で結合して表示する。例: `[--|**]name`。

## bash


```bash
#!/bin/bash

command -v tmux &>/dev/null || exit 0

# Skip when running inside session_summarizer subprocess
[ "$CLAUDE_SUMMARIZER_RUNNING" = "1" ] && exit 0

action="$1"
pane="$TMUX_PANE"

if [ -z "$pane" ]; then
  exit 0
fi

# --- 0. Skip if pane already has the same status ---
current=$(tmux show-options -p -t "$pane" -v @claude-status 2>/dev/null)
if [ "$current" = "$action" ]; then
  exit 0
fi

# --- 1. Update per-pane status ---
case "$action" in
  start | thinking | done | notification)
    tmux set-option -p -t "$pane" @claude-status "$action"
    ;;
  reset)
    tmux set-option -pu -t "$pane" @claude-status
    ;;
  *)
    echo "tmux-window-claude-status: unknown action '$action'" >&2
    exit 1
    ;;
esac

# --- 2. Collect statuses from all panes in the same window ---
window=$(tmux display-message -t "$pane" -p '#{window_id}')

symbols=()
has_thinking=false
has_notification=false
has_active=false

while IFS= read -r pane_id; do
  status=$(tmux show-options -p -t "$pane_id" -v @claude-status 2> /dev/null)
  case "$status" in
    start)
      symbols+=("--")
      has_active=true
      ;;
    thinking)
      symbols+=("**")
      has_thinking=true
      has_active=true
      ;;
    done)
      symbols+=("==")
      has_active=true
      ;;
    notification)
      symbols+=("??")
      has_notification=true
      has_active=true
      ;;
  esac
done < <(tmux list-panes -t "$window" -F '#{pane_id}')

# --- 3. Update window name and style ---
if [ "$has_active" = true ]; then
  # Save original window name before first Claude rename
  saved=$(tmux show-options -wv -t "$window" @saved-window-name 2>/dev/null)
  if [ -z "$saved" ]; then
    saved=$(tmux display-message -t "$window" -p '#W')
    tmux set-option -w -t "$window" @saved-window-name "$saved"
  fi

  joined=$(
    IFS='|'
    echo "${symbols[*]}"
  )
  tmux rename-window -t "$window" "[${joined}]${saved}"

  if [ "$has_notification" = true ]; then
    tmux set-option -w -t "$window" window-status-style 'fg=yellow,bold'
  elif [ "$has_thinking" = true ]; then
    tmux set-option -w -t "$window" window-status-style 'fg=blue,bold'
  else
    tmux set-option -w -t "$window" window-status-style 'fg=green,bold'
  fi
else
  # All panes inactive: restore saved window name
  saved=$(tmux show-options -wv -t "$window" @saved-window-name 2>/dev/null)
  if [ -n "$saved" ]; then
    tmux rename-window -t "$window" "$saved"
    tmux set-option -wu -t "$window" @saved-window-name
  fi
  tmux set-option -wu -t "$window" window-status-style
fi
```

`tmux set-option -p`で各ペインに `@claude-status` を保持。次に `tmux list-panes` で同一ウィンドウの全ペインを走査してステータスを集約し、最後にウィンドウ名とスタイルをまとめて更新。

色の優先順位は `notification` (黄) > `thinking` (青) > `done/start` (緑)。

いくつか最適化を入れている：
- tmux が無い環境では早期終了
- `CLAUDE_SUMMARIZER_RUNNING=1` のときはスキップ（セッションサマリー生成時の不要な更新を防ぐ）
- 同じステータスならスキップ（無駄な tmux コールを減らす）

また、元のウィンドウ名を `@saved-window-name` に保存しておき、セッション終了時に復元するようにしている。以前は `automatic-rename on` に戻していたが、自分で名前を設定しているウィンドウだと戻ってしまうので、保存・復元方式に変更した。


## settings.json

```json
"hooks": {
  "SessionStart": [
    {
      "matcher": "",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh start" }]
    }
  ],
  "UserPromptSubmit": [
    {
      "matcher": "",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh thinking" }]
    }
  ],
  "PostToolUse": [
    {
      "matcher": "",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh thinking" }]
    }
  ],
  "Notification": [
    {
      "matcher": "permission_prompt",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh notification" }]
    }
  ],
  "Stop": [
    {
      "matcher": "",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh done" }]
    }
  ],
  "SessionEnd": [
    {
      "matcher": "",
      "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh reset" }]
    }
  ]
}
```

`PostToolUse` でツール使用中も `thinking` 状態を維持。`Notification` は `permission_prompt` にマッチさせ、権限確認待ちのときに黄色で通知。`reset` では保存していたウィンドウ名を復元し、スタイルを解除。

## おわり

処理待ちの間別のウィンドウやペインで別の作業して、その間に別のウィンドウの思考が終わって、という、行ったり来たりをすることが最近増えており、まぁまぁ便利になりました。
