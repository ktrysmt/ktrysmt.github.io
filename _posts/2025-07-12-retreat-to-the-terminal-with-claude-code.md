---
layout: post
title: "claude-codeのおかげでterminal引きこもり生活に舞い戻る"
date: 2025-07-12 09:00:00 +0900
categories: LLM
published: true
use_toc: false
---



[前回の記事](https://ktrysmt.github.io/blog/summary-of-my-recent-llm-activities/)から少し変化があったので更新

`roo code`のrole切り替えやorchestratorが使いやすかったので長らく使っていましたが、terminalからvscodeに切り替えないといけないこととvscodeということでどうしてもGUIを要求されがちなこと、キーバインド設定が大変なことが不満でした。

この辺の小さなストレスが`claude-code`によってだいぶ解消し、terminal生活が再開しました。おかげで最近はもう技術系タスクは`claude-code`一択です。

`claude-code`向けにちょこちょこと調整した箇所があるので、それらを踏まえ今のところの状況をまとめておきます。

## vim plugin

- <https://github.com/greggh/claude-code.nvim>
- <https://github.com/yetone/avante.nvim>
- <https://github.com/kiddos/gemini.nvim>
- <https://github.com/zbirenbaum/copilot.lua>

`claude-code.nvim`はあまり使わないけど自身でもファイルを開いていて手直しをしたいであろうときに使います。このプラグイン経由だとclaudeが更新したファイルの内容をバッファに反映してくれるのがよいです。
`avante.nvim`はビジュアルモードで選択してaskしたりeditしたりが便利なのでそこだけ使ってます。シグネチャのすぐ上にコメント書いて`gemini`なり`copilot`で補完させてもいいですが、`avante.nvim`にはclaudeのmodelを当ててるので確率を高めたいときは`avante.nvim`、とりあえずだせればOKなら`gemini`か`copilot`、みたいにして使い分けてます。

キーマップとコマンドの組み合わせをちゃんとやれば依存プラグインをもうちょっと減らせそうですが、苦労の割にリターンが薄いので一旦雑に上記でやってます。

## terminal keybind
1. `shift + enter`を`meta + enter`にremap
2. `ctrl + space`、`esc`および`ctrl + c`を入力したらその直前に強制IME英数モードに切り替えてから元のキーバインドを改めて入力

1.の`meta + enter`は`claude-code`限定なのでterminal側で設定してます。なおterminalは[wezterm](https://wezterm.org/)を愛用してます。
2.については処理がやや複雑なのでwindowsの場合は上記をkeyhac、macosの場合はkarabiner-elementsで設定してます。

wezterm(windows/macos共通)の例
```lua
local wezterm = require 'wezterm';
local act = wezterm.action
return {
  keys = {
    { key = 'Enter', mods = 'SHIFT',        action = act.SendKey { key = 'Enter', mods = 'META' } },
  }
}
```

keyhacの例
```python
keymap_wez = keymap.defineWindowKeymap(
    exe_name="wezterm-gui.exe", class_name="org.wezfurlong.wezterm")
def wezterm_ctrl_c():
    keymap.getWindow().setImeStatus(0)
    keymap.InputKeyCommand("LCtrl-C")()
keymap_wez[ "LCtrl-C" ] = wezterm_ctrl_c
def wezterm_ctrl_space():
    keymap.getWindow().setImeStatus(0)
    keymap.InputKeyCommand("LCtrl-Space")()
keymap_wez[ "LCtrl-Space" ] = wezterm_ctrl_space
```

karabiner-elementsの例
```json
{
  "type" : "basic",
  "from": { "key_code": "c", "modifiers": {"mandatory": [ "left_control" ]} },
  "to": [ { "key_code": "japanese_eisuu" },{ "key_code": "c", "modifiers":  "left_control" } ],
  "conditions": [{
    "type": "input_source_if",
    "input_sources": [{ "language": "ja" }]
  },
  {
    "type": "frontmost_application_if",
    "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.github\\.wez\\.wezterm$"]
  }]
},
{
  "type" : "basic",
  "from": { "key_code": "spacebar", "modifiers": {"mandatory": [ "left_control" ]} },
  "to": [ { "key_code": "japanese_eisuu" },{ "key_code": "spacebar", "modifiers":  "left_control" } ],
  "conditions": [{
    "type": "input_source_if",
    "input_sources": [{ "language": "ja" }]
  },
  {
    "type": "frontmost_application_if",
    "bundle_identifiers": ["^com\\.apple\\.Terminal$", "^com\\.github\\.wez\\.wezterm$"]
  }]
}
```

## mcp

- awslabs.aws-documentation-mcp-server@latest
- @choplin/mcp-gemini-cli
- mcp-think-as
- mcp-deepwiki@latest
- @upstash/context7-mcp
- @playwright/mcp@latest
- arxiv-mcp-server

最近良く使うのはこの辺。モデルの性能や`claude-code`の機能のおかげで小規模な自作mcpは軒並み不要になりつつあります。

## custom slash command

### /gemini-search
- <https://github.com/ktrysmt/dotfiles/blob/master/.claude/commands/gemini-search.md>

[元ネタ](https://zenn.dev/mizchi/articles/gemini-cli-for-google-search)を少し改変して`$ARGUMENTS`について明記してます。

### /tdd
- <https://github.com/ktrysmt/dotfiles/blob/master/.claude/commands/tdd.md>

こちらも[元ネタ](https://github.com/KentBeck/BPlusTree3/blob/main/rust/docs/CLAUDE.md)を改変して`$ARGUMENTS`の有無で少し指示が変わるように変更。TDDに寄せたシステムプロンプトの調整には長らく苦戦してきましたが、これとclaude系モデルの組み合わせだとまぁまぁワークしてくれました。

### /think-with-4d
- <https://github.com/ktrysmt/dotfiles/blob/master/.claude/commands/think-with-4d.md>

4d methodology が汎用的に使えるっぽいのをredditかなにかで見かけたので色々比較しながらできるだけ抽象的に記述してます。gptとやり取りするとき特有の2手先を読むような思考系タスクのときのreasoningの中身を覗くと、わりと意図通りのthinking logがでてくれているのを確認できました。claudeで使うときは`roo code`のarchitectロールと同じような感じで、やや不確実性が残る状態でスタートするときの前処理として考察の経緯をコンテキストに残させるとき使っています。拡張思考モードになるように明示的に指示した限りではgptと同様thinking logにそれっぽい順序で出力していたし合意できる点が多かったので、まぁまぁワークしてそうです。

## モデルごとの使い分け

* o3, o3 pro: 計画を立てたり不確実性の高いことを考えたり、トレードオフなテーマの詳細を詰めたり
* sonnet, opus: コードベースを元にした作業全般。アーキテクチャ、コード、テスト
* grok: 鮮度が重要な情報を探索するときの初手、各種deep research系より気軽で性能もいいので重宝してます
* gemini 2.5 pro: 検索、またはワンショットで手早くそれなりの回答を得たいときにとりあえず裏で投げておく、最近あまり使ってないかも

