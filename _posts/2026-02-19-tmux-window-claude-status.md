---
layout: post
title: "ClaudeCodeの進捗状況をtmuxウィンドウ名に反映する"
date: 2026-02-19 23:00:00 +0900
categories: Tmux
published: true
use_toc: false
---


ClaudeCode を tmux で使っていると別ウィンドウに切り替えたとき「今あっちの Claude はどうなってるかわかりにくい」問題。これを解消したい。


## 挙動

| 状態 | ウィンドウ名 | 色 |
|---|---|---|
| Claudeセッション開始 | `[--]name` | デフォルト |
| Claude考え中 | `[**]name` | 青・太字 |
| Claude応答完了 | `[==]name` | 緑・太字 |
| Claudeセッション終了 | `name` (自動) | デフォルト |

prefix `[--]` `[**]` `[==]` をつけるだけだと視認性がイマイチなので色を付けるのも同時にやるようにしているが、色かprefixどっちかでもいいかも。

同じウィンドウ内に複数ペインで Claude を起動している場合は、各ペインのシンボルを `|` で結合して表示する。例: `[--|**]name`。

## bash


```bash
#!/bin/bash

action="$1"
pane="$TMUX_PANE"

if [ -z "$pane" ]; then
  exit 0
fi

# --- 1. Update per-pane status ---
case "$action" in
  start|thinking|done)
    tmux set-option -p -t "$pane" @claude-status "$action"
    ;;
  reset)
    tmux set-option -pu -t "$pane" @claude-status
    ;;
esac

# --- 2. Collect statuses from all panes in the same window ---
window=$(tmux display-message -t "$pane" -p '#{window_id}')
dir=$(basename "$(tmux display-message -t "$pane" -p '#{pane_current_path}')")

symbols=()
has_thinking=false
has_active=false

while IFS= read -r pane_id; do
  status=$(tmux show-options -p -t "$pane_id" -v @claude-status 2>/dev/null)
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
  esac
done < <(tmux list-panes -t "$window" -F '#{pane_id}')

# --- 3. Update window name and style ---
if [ "$has_active" = true ]; then
  joined=$(IFS='|'; echo "${symbols[*]}")
  tmux rename-window -t "$window" "[${joined}]${dir}"

  if [ "$has_thinking" = true ]; then
    tmux set-option -w -t "$window" window-status-style 'fg=blue,bold'
  else
    tmux set-option -w -t "$window" window-status-style 'fg=green,bold'
  fi
else
  # All panes inactive: restore automatic rename
  tmux set-option -w -t "$window" automatic-rename on
  tmux set-option -wu -t "$window" window-status-style
fi
```

`tmux set-option -p`で各ペインに `@claude-status` を保持。次に `tmux list-panes` で同一ウィンドウの全ペインを走査してステータスを集約し、最後にウィンドウ名とスタイルをまとめて更新。いずれかのペインが `thinking` 状態なら青、全ペインが `done` or `start` なら緑に。


## settings.json

```json
"hooks": {
  "SessionStart": [
    { "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh start" }] }
  ],
  "UserPromptSubmit": [
    { "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh thinking" }] }
  ],
  "Stop": [
    { "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh done" }] }
  ],
  "SessionEnd": [
    { "hooks": [{ "type": "command", "command": "bash ~/.claude/hooks/tmux-window-claude-status.sh reset" }] }
  ]
}
```

`reset` では `automatic-rename on` に戻してウィンドウ名の制御を tmux に返してる。`-wu` フラグでスタイルをunset。

## おわり

処理待ちの間別のウィンドウやペインで別の作業して、その間に別のウィンドウの思考が終わって、という、行ったり来たりをすることが最近増えており、まぁまぁ便利になりました。
