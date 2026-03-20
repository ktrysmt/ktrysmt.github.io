---
layout: post
title: "bufを使ってgRPCのmockを建てる"
date: 2026-03-21 00:10:00 +0900
categories: [Cloud Infrastructure]
published: true
use_toc: false
description: "buf buildの出力をそのまま入力にしてgRPCモックサーバーを即座に立ち上げるFauxRPCの導入手順まとめ。Stub定義など"
tags:
  - grpc
  - devtools
  - protobuf
---

`buf` で Protocol Buffers を管理しているプロジェクトで、`.proto` から手軽に gRPC モックサーバーを建てたかったのだが。いくつか調べた結果、buf エコシステムとの親和性が高そうな FauxRPC を使ってみることに。`buf build` + `fauxrpc run` の 2 ステップ。

## FauxRPC

Protobuf 定義からフェイクの gRPC / gRPC-Web / Connect / REST サーバーを上げてくれる。`buf build` の出力 (ディスクリプタ) をそのまま入力にできる。

- buf ネイティブ
- gRPC / gRPC-Web / Connect / REST
- Server Reflection 有効 (grpcurl でもそのまま叩ける)
- Stub 定義で固定レスポンス
- protovalidate 連携によるバリデーションルール準拠のデータ生成
- コード生成・実装が一切不要
- HTTP/1.1 (Connect) と HTTP/2 (gRPC) の両方をサポートしているので `grpcurl`, `buf curl` いずれも使える

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

FauxRPC (v0.19.5) - 1 services loaded, 0 stubs loaded
Listening on http://:18081
OpenAPI documentation: http://:18081/fauxrpc/openapi.html
```


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

## Makefile例

```makefile
mock/grpc: libs/proto/service.binpb
	fauxrpc run --schema=$< --stubs=stubs/ --addr=:18081 --dashboard

libs/proto/service.binpb: $(wildcard libs/proto/**/*.proto)
	@cd libs/proto && buf build -o service.binpb
```

## 注意点

- フェイクデータは型ベースのランダム値なので、`status` enum などがビジネスロジック上ありえない値になりうるる。重要なテストシナリオには Stub 定義が必須
- `int64` フィールドに非常に大きなランダム値が入る。現実的な値が必要ならやはり Stub か protovalidate の制約

## チーム間での共有

protoとMakefileなどがリポジトリにあれば十分で、各自の手元で `buf build` からモック起動まで完結できる。前述の Makefile 例のように `make mock/grpc` 一発で済む。

`--schema` は複数指定できるので、マイクロサービス構成でも `--schema=a.binpb --schema=b.binpb` のように合成可能。

## おわり

まぁまぁ体験よいです。

## 参考

- [FauxRPC 公式ドキュメント](https://fauxrpc.com/docs/intro/)
- [FauxRPC GitHub](https://github.com/sudorandom/fauxrpc)
- [FauxRPC Input Sources](https://fauxrpc.com/docs/inputs/)
- [FauxRPC と Protovalidate](https://kmcd.dev/posts/fauxrpc-protovalidate/)
