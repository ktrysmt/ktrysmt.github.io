---
layout: post
title: "MCP Script Runner: スクリプトを雑にmcpサーバのツールにするやつ"
date: 2025-04-03 22:30:00 +0900
categories: LLM
published: true
use_toc: false
description: "シンプルかつ自由度高めのコマンド実行サーバーを作ったので解説"
---


[](https://github.com/ktrysmt/mcp-script-runner){:.card-preview}

MCP Script Runner は主にシェルスクリプトなどを簡単にサーバーツール化します。
似たようなことをしてるひとは多いですが使いやすい雛形が欲しかったのでmcpの理解ついでに作りました。

## 機能

1. コマンド実行（`exec_command`）
2. コマンド一覧取得（`list_commands`）

## 前提

- python
- uv

## 設定

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

