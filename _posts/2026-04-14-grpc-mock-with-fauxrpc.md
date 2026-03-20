---
layout: post
title: "buf起点でgRPCモックサーバーを気軽に建てる -- FauxRPC実践"
date: 2026-04-14 09:00:00 +0900
categories: Cloud Infrastructure
published: true
use_toc: false
description: "buf buildの出力をそのまま入力にしてgRPCモックサーバーを即座に立ち上げるFauxRPCの導入手順。Stub定義やMakefile統合など実践的な活用方法も紹介。"
tags:
  - grpc
  - devtools
  - protobuf
---

`buf` で Protocol Buffers を管理しているプロジェクトで、`.proto` から手軽に gRPC モックサーバーを建てたかった。いくつか調べた結果、buf エコシステムとの親和性が一番高い FauxRPC を選択。`buf build` + `fauxrpc run` の 2 ステップで終わるのでかなりラク。

## FauxRPC とは

Protobuf 定義からフェイクの gRPC / gRPC-Web / Connect / REST サーバーを即座に立ち上げるツール。`buf build` の出力 (ディスクリプタ) をそのまま入力にできるため、buf ユーザーにとって自然な選択肢になる。

主な特長:

- buf ネイティブ (binpb / BSR 直接対応)
- gRPC / gRPC-Web / Connect / REST マルチプロトコル対応
- Server Reflection 有効 (grpcurl でそのまま叩ける)
- Stub 定義で固定レスポンスを返せる (YAML/JSON、CEL 式対応)
- protovalidate 連携によるバリデーションルール準拠のデータ生成
- コード生成・実装が一切不要

## セットアップ

### 1. インストール

```bash
go install github.com/sudorandom/fauxrpc/cmd/fauxrpc@latest
```

### 2. ディスクリプタ生成

```bash
cd libs/proto
buf build -o service.binpb
```

既存の `buf.yaml` がそのまま使われるため、追加設定は不要。

### 3. モックサーバー起動

```bash
fauxrpc run --schema=service.binpb --addr=:18081
```

```
FauxRPC (v0.19.5) - 1 services loaded, 0 stubs loaded
Listening on http://:18081
OpenAPI documentation: http://:18081/fauxrpc/openapi.html
```

たった 3 コマンドでモックサーバーの起動まで完了する。

## 動作確認

### メソッド一覧の取得

```bash
$ buf curl --http2-prior-knowledge http://localhost:18081 --list-methods
my.package.v1.MyService/MethodA
my.package.v1.MyService/MethodB
my.package.v1.MyService/MethodC
...
```

### grpcurl での呼び出し

Server Reflection 有効のため、grpcurl からもそのまま使える。

```bash
# サービス一覧
$ grpcurl -plaintext localhost:18081 list
my.package.v1.MyService

# RPC 呼び出し (リクエストボディなし)
$ grpcurl -plaintext -d '{}' localhost:18081 my.package.v1.MyService/MethodA
{
  "healthy": true,
  "message": "Practice hand drills regularly."
}

# RPC 呼び出し (リクエストボディあり)
$ grpcurl -plaintext \
  -d '{"field1":"value1","field2":"value2"}' \
  localhost:18081 my.package.v1.MyService/MethodB
{
  "status": "RESPONSE_STATUS_ERROR_MAINTENANCE",
  "errorMessage": "Reduce cognitive load in the eye.",
  "serverTime": "1992-05-18T16:07:05.724224150Z"
}
```

### レスポンスの特徴

スキーマの型情報に基づくランダムなフェイクデータが自動生成される。

- `string` -- ランダムな英文
- `bool` -- true / false
- `int64` -- ランダムな整数値
- `enum` -- 定義された値からランダム選択
- `message` (nested) -- 再帰的にフェイクデータ生成
- `google.protobuf.Timestamp` -- ランダムな日時

ネストされたメッセージも含め完全自動:

```json
{
  "apiVersion": "1.8.20",
  "status": "RESPONSE_STATUS_ERROR_RATE_LIMITED",
  "configuration": {
    "pollIntervalSeconds": "8783008286415803127",
    "recordsBatchSize": "1999146540591599958",
    "listsSyncIntervalSeconds": "4314275503481523751",
    "configVersion": "1094785172391076840"
  }
}
```

## Stub による固定レスポンス

デフォルトのランダムデータでは不十分な場合、Stub ファイルで特定 RPC への固定レスポンスを定義できる。

```yaml
# stubs/method_a.yaml
stubs:
  - service: my.package.v1.MyService
    method: MethodA
    response:
      content:
        healthy: true
        message: "ok"
```

```bash
fauxrpc run --schema=service.binpb --stubs=stubs/ --addr=:18081
```

## Makefile 統合例

```makefile
mock/grpc: libs/proto/service.binpb
	fauxrpc run --schema=$< --stubs=stubs/ --addr=:18081 --dashboard

libs/proto/service.binpb: $(wildcard libs/proto/**/*.proto)
	@cd libs/proto && buf build -o service.binpb
```

## 注意点

- フェイクデータは型ベースのランダム値なので、`status` enum がビジネスロジック上ありえない値になることもある。重要なテストシナリオには Stub 定義が必要
- `int64` フィールドに非常に大きなランダム値が入る。現実的な値が必要なら Stub か protovalidate の制約で対応
- HTTP/1.1 (Connect) と HTTP/2 (gRPC) の両方をサポートしているので `grpcurl` も `buf curl` も使える

## 所感

proto ファイル変更時はディスクリプタ再生成だけでモックも追従するのが良い。フロントエンド開発やクライアントの結合テストで、バックエンド実装待ちが不要になる。

BSR (Buf Schema Registry) に Push 済みなら `fauxrpc run --schema=buf.build/your-org/your-repo` でチーム全員が同一モックを即座に起動できるのも便利。

## 参考

- [FauxRPC 公式ドキュメント](https://fauxrpc.com/docs/intro/)
- [FauxRPC GitHub](https://github.com/sudorandom/fauxrpc)
- [FauxRPC Input Sources](https://fauxrpc.com/docs/inputs/)
- [FauxRPC と Protovalidate](https://kmcd.dev/posts/fauxrpc-protovalidate/)
