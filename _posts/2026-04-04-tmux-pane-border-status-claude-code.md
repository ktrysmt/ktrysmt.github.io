---
layout: post
title: "ClaudeCode teamsがtmuxのpane-border-statusをtopに変えてしまう問題の対処"
date: 2026-04-04 12:00:00 +0900
categories: [Developer Tools]
published: true
description: "ClaudeCode teamsでteammateを起動するとtmuxのpane-border-statusがbottomからtopに変わる問題。ClaudeCodeのstatusline機能がpane/windowレベルでtopを設定しグローバル設定を上書きしていたため、set-hookとClaudeCode hookの両方で対処"
tags:
  - tmux
  - claude-code
  - hooks
---

ClaudeCode teams で teammate を起動すると、`.tmux.conf` で指定していた `pane-border-status bottom` が `top` に変わってしまう。

ClaudeCode の statusline 機能が、ペインレベル (`-p`) とウィンドウレベル (`-w`) の両方で `pane-border-status top` を設定していた。
tmux のオプション解決は pane > window > global の順で優先されるため、グローバル (`-g`) で `bottom` を指定していてもペイン/ウィンドウレベルの `top` で上書き。

なので2箇所で `bottom` を強制。

### 1. .tmux.conf に set-hook を追加

ウィンドウやペインが新規作成されるタイミングで、ウィンドウレベルの `pane-border-status` を `bottom` に強制する。

```tmux
set-hook -g after-split-window "set-option -w pane-border-status bottom"
set-hook -g after-new-window "set-option -w pane-border-status bottom"
```

### 2. ClaudeCode の hook スクリプトで再設定

ClaudeCode の hook (UserPromptSubmit, PostToolUse, Notification, Stop) が発火するたびに、ペイン/ウィンドウレベルで `bottom` を再設定する。

statuslineのhookで走らせるshellにて

```bash
tmux set-option -p -t "$pane" pane-border-status bottom 2>/dev/null
```

`-p` (ペインレベル) と `-w` (ウィンドウレベル) の両方で設定することで、ClaudeCode の statusline が後からどちらのレベルで `top` を設定しても `bottom` で上書き。

