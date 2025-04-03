---
layout: post
title: "スクリプトを雑にmcpサーバのツールにするやつ"
date: 2025-04-03 22:30:00 +0900
categories: LLM
published: true
use_toc: false
description: ""
---

[](https://github.com/ktrysmt/mcp-script-runner){:.card-preview}

主にシェルスクリプトなどをずぼらにサーバーツール化するmcpサーバです。
似たようなことをしてるひとは多いでしょうが使いやすい雛形が欲しかったのでmcp理解ついでに作成。

### 機能

1. コマンド実行（`exec_command`）
2. コマンド一覧取得（`list_commands`）

### 前提

- python
- uv

### 設定

- 基本はmain.py実行でOK
- main.pyと同じ場所にcommands/フォルダがあるので、そこにツール化したいスクリプトを置くだけ
- commandsフォルダは環境変数 `COMMAND_DIRECTORY` で変更可能
- dotenv対応、main.pyと同じ場所に.envを配置


READMEそのままですがmcpの設定例は以下

```json
{
  "mcpServers": {
    "mcp-script-runner": {
      "command": "uv",
      "args": [
        "--directory",
        "/path/to/mcp-script-runner",
        "run",
        "main.py"
      ]
    }
  }
}
```

### 注意点

* ファイルを直接実行するので各ファイルにはシェバンを忘れないこと。
* ファイルには実行権限が必要（一応permissionを調整する処理を入れてあるが）。
* あくまでローカルでの利用および自分が把握しているスクリプトのみを置くこと。

