---
layout: post
title: "アウトバウンドを制限しながら VPC Endpoint を使う"
date: 2022-02-18 00:00:00 +0900
comments: true
categories: "AWS"
published: true
use_toc: false
description: ""
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

アウトバウンド閉じる意味はDNSトンネリングのせいで薄れているなと長らく思っていましたが、trailを確認したりあとからフォレンジックする際にノイズが減って特定が早まる可能性があるので、やっておく価値はあるなと最近気持ちを改めました。