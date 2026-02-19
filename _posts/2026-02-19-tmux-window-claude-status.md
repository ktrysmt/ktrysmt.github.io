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

## bash


```bash
action="$1"
pane="$TMUX_PANE"

if [ -z "$pane" ]; then
  exit 0
fi

dir=$(basename "$(tmux display-message -t "$pane" -p '#{pane_current_path}')")

case "$action" in
  start)
    tmux rename-window -t "$pane" "[--]${dir}"
    ;;
  thinking)
    tmux rename-window -t "$pane" "[**]${dir}"
    tmux set-option -w -t "$pane" window-status-style 'fg=blue,bold'
    ;;
  done)
    tmux rename-window -t "$pane" "[==]${dir}"
    tmux set-option -w -t "$pane" window-status-style 'fg=green,bold'
    ;;
  reset)
    tmux set-option -w -t "$pane" automatic-rename on
    tmux set-option -wu -t "$pane" window-status-style
    ;;
esac
```

`$TMUX_PANE` は tmux がペインに自動でセットする環境変数。これを使って `tmux rename-window` と `tmux set-option` を叩く。


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
