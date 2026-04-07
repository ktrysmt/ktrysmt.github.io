---
layout: post
title: "gh issue createのGraphQL権限エラーはREST APIで回避した"
date: 2026-03-27 06:00:00 +0900
categories: [Cloud Infrastructure]
published: true
description: "GitHub Actionsのgh issue createが「Resource not accessible by integration」で失敗する問題。permissions設定が正しくてもGraphQLの権限チェックで弾かれるケースがあり、gh apiによるREST API直接呼び出しで回避する"
tags:
  - github-actions
  - gh-cli
  - rest-api
---

GitHub Actions で `gh issue create` を使おうとしたら、`permissions: issues: write` などをちゃんとを設定しているのに以下のエラーで失敗する件です。

```
GraphQL: Resource not accessible by integration (createIssue.issue)
```

原因と回避策の備忘録です。

## 状況

- プライベートリポジトリ
- `GITHUB_TOKEN` (自動生成トークン) を使用
- `workflow_dispatch` および `schedule` トリガー

## 発生したエラー

ワークフローで `issues: write` 権限を明示的に付与し、リポジトリ設定でも `Read and write permissions` を選択しているにもかかわらず、`gh issue create` が失敗します。

```yaml
permissions:
  contents: read
  issues: write
  id-token: write

jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - name: Create issue
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh issue create \
            --title "タイトル" \
            --body-file /tmp/report.md \
            --label "my-label"
```

## 原因

`gh issue create` は内部的に GitHub GraphQL API を使っています。この GraphQL の `createIssue` mutation に対して `GITHUB_TOKEN` ではアクセスを拒否されてしまいます。

ひと通り確認した項目:

| 確認項目 | 結果 |
|---|---|
| ワークフローの `permissions` に `issues: write` | 設定済み |
| リポジトリ Settings > Actions > Workflow permissions | Read and write |
| Allow GitHub Actions to create and approve pull requests | 有効 |
| 前段のアクションによるトークン上書き | なし |
| `GH_TOKEN` 環境変数での明示的なトークン渡し | 設定済み |

全て問題ないのに GraphQL 経由でだけ権限エラーが出ます。GraphQL API と REST API で `GITHUB_TOKEN` の権限チェックに差異がある模様です。GraphQLのほうが仕組み的にアグリゲート等行う関係上追加で必要な権限が（repo系ではなくユーザー系で）ありそうな気がします。

## 回避策: REST API を使う

`gh issue create` (GraphQL) の代わりに `gh api` で REST API を直接呼ぶと必要な権限はピンポイントで定まるので、安定しました。

```yaml
      - name: Create issue
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh api repos/${{ github.repository }}/issues \
            -f title="タイトル" \
            -f body=@/tmp/report.md \
            -f "labels[]=my-label"
```

- `gh api` は REST API (`POST /repos/{owner}/{repo}/issues`) を使います
- `-f body=@<file>` でファイルからボディを読み込めます (`@` プレフィックスがファイル読み込みの意味)
- ラベルは `-f "labels[]=label-name"` の形式で配列として渡せます
- 複数ラベルを付ける場合は `-f "labels[]=label1" -f "labels[]=label2"` と繰り返します

なおラベルが存在しない場合は事前に作成しておきます。`gh label create` は REST API ベースなので影響を受けません。

```yaml
          gh label create my-label \
            --description "説明" \
            --color 1D76DB \
            --force 2>/dev/null || true
```

## 補足: gh CLI サブコマンドと内部 API の対応

| サブコマンド | 内部 API | 備考 |
|---|---|---|
| `gh issue create` | GraphQL | 今回問題のやつ |
| `gh issue list` | GraphQL | 同様の問題が起きうる |
| `gh api repos/.../issues` | REST | 回避策として利用 |
| `gh label create` | REST | 影響なし |
| `gh pr create` | GraphQL | 同様の問題が起きうる |

## おわり

権限まわりでハマったときはRESTに置き換えるとよいかもしれません。

