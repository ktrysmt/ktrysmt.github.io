---
layout: post
title: "pnpmで依存パッケージに一時的なpatchを当てる"
date: 2026-03-22 12:00:00 +0900
categories: [Developer Tools]
published: true
use_toc: false
description: "npm公開とかしてるとたまに必要になる、依存パッケージにpatchを当てる話。"
tags:
  - pnpm
  - nodejs
---

依存パッケージにバグや不足機能があるとして upstream に PR を出すほどでもない、あるいは PR がマージされるまで待てないってときに。

## 手順

### 1. patch 対象を展開する

```bash
pnpm patch @ktrysmt/beautiful-mermaid@1.3.0
```

一時ディレクトリにパッケージが展開される。パスが標準出力に出る。

### 2. コードを編集する

展開されたディレクトリ内のファイルを直接編集する。

```bash
vim /tmp/xxxxx/dist/index.js
```

ビルド済みの `dist/` を触ることになるので、変更箇所は最小限にする。

### 3. patch を確定する

```bash
pnpm patch-commit /tmp/xxxxx
```

`patches/` ディレクトリに `.patch` ファイルが生成され `package.json` に `pnpm.patchedDependencies` が自動追加される。

```json
{
  "pnpm": {
    "patchedDependencies": {
      "@ktrysmt/beautiful-mermaid@1.3.0": "patches/@ktrysmt__beautiful-mermaid@1.3.0.patch"
    }
  }
}
```

### 4. install し直す

```bash
pnpm install
```

patch が適用された状態で `node_modules` に展開される。

### 5. 動作確認

```bash
pnpm test
```

### 6. patch が不要になったら

upstream に修正が取り込まれたら、`pnpm.patchedDependencies` を `package.json` から消して `patches/` 内の `.patch` ファイルを削除する。その後 `pnpm install` で元に戻る。

## 注意

- patch はバージョンに紐づくようで依存パッケージをアップデートしたら patch が当たらなくなる。バージョンを上げるたびに patch を作り直すか都度確認。
- `dist/` いじってるだけなのであくまで一時的な措置。
- もしチームでこれを使う場合は `.patch` ファイルと `package.json` の変更をコミットしておいて `pnpm install` すれば、他のメンバーにも同じ patch が当たってくれる。

## pnpm link でローカルテスト

patch 前にローカルで動くか確認したいときは `pnpm link`

```bash
# ローカルのパッケージを直接リンク
pnpm link /path/to/beautiful-mermaid

# テスト
pnpm test

# リンク解除して registry 版に戻す
rm node_modules/@ktrysmt/beautiful-mermaid
pnpm install
```

`pnpm link` は `node_modules` にsymlink張るだけなのでローカルで編集した内容は当然即反映。

## おわり

めんどいけど `yarn patch` より少し手間が少ないっぽくて、一時的にpatch運用を受け入れるとするなら体験はいいほうだったと思う。

