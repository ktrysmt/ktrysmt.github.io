---
layout: post
title: "アウトバウンドを制限しながら VPC Endpoint を使う"
date: 2022-02-17 23:55:00 +0900
comments: true
categories: [Cloud Infrastructure]
published: true
description: "アウトバウンドを閉じた環境でVPC Endpoint Interface Typeを使う手順と注意点をまとめます。SGのegress設定やGateway Typeとの違いも解説。"
tags:
  - aws
  - security
  - infrastructure
---

地味にハマったのでメモです。

## 要件

* 443などのアウトバウンドを閉じて、あるいはprivate-subnetを使いつつもNATは建てずになんらかのコンピューティングを利用している
* vpc endpoint interface type を設定できる

## 手順

1. 任意のサービスの vpc endpoint interface type を作成する
2. 作成完了まで待つと対応する eni が判明する
3. eni には作成した subnet の数だけ private ip 払い出されるので ip をメモ
4. 任意のコンピューティングの SG の egress に3.でメモした ip を https(tcp:443) として設定

## 注意点

* vpc に対し vpc endpoint interface type を設定する際、subnet が private か public かは実は問わない、azに対し１つずつ設定できる。同 vpc の同 az に対しpublic/privateそれぞれ（ダブって）設定しようとするとエラーが返る。
* gateway type (s3,dynamo) ではこの方式は使えず、公式に提供されている prefixlist を使って SG egress に設定することで要件は満たせる。

アウトバウンド閉じる意味はDNSトンネリングのせいで薄れているなと長らく思っていましたが、trailを確認したりあとからフォレンジックする際にノイズが減って特定が早まる可能性があるので、やっておく価値はあるなと最近認識を改めました。

## 関連記事

- [VPC内部通信の暗号化はなぜ必要か](/blog/internal-encryption-fsa-gl-fisc/) -- FSA GL/FISCの規制要件と内部通信暗号化
- [SSE-S3とSSE-KMSの違いを知る](/blog/difference-between-sse-s3-and-sse-kms/) -- S3暗号化方式の選定基準
- [S3とCloudTrailとKMS暗号化の、ちょっと複雑な関係について](/blog/s3-and-cloudtrail-and-kms/) -- S3/CloudTrail/KMSの暗号化の関係性
