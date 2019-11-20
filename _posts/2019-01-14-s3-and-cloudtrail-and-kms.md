---
layout: post
title: 'S3とCloudTrailとKMS暗号化の、ちょっと複雑な関係について'
date: 2019-01-14
comments: true
categories: AWS
published: true
use_toc: false
description: 'AWSのS3・CloudTrail・KMSはいろいろと入り組んだ？関係性になっていて、初見でもろもろをすんなり理解することはちょっと難しそうかなと思い、備忘録です。'
---

わかりにくい箇所をポイントごとにまとめました。

## S3のObjectLevelLogging

いくつかありますが特にわかりにくいのはS3の機能のうちの ObjectLevelLogging かなと思ってます。

ObjectLevelLogging はマネコン上ではあたかもS3が提供する機能のようにみえますが、実態はCloudTrailで提供される機能で実現されます。

なお、CloudTrailでのObjectLevelLoggingは、

* `DataResources:` に `AWS::S3::Object` （とBucketのパス）を指定
* `IncludeManagementEvents` を `false` に設定

と、することで実現できます。


## CloudTrail側の暗号化とSSE-S3

CloudTrail側でEncryptionをNoにすると、CloudTrailはSSE-S3を使いつつCloudTrail保存先として指定されるS3Bucketに、証跡をPutします。
なので、EncryptionはNoに設定しましたが、厳密にはEncryptionはNoではないのです。

これはCloudTrailのデフォルトの挙動で、変更できないようです。

なお、CloudTrail設定時にダイジェストファイルの作成オプションがありますが、これを有効化した場合、このダイジェストファイルの生成にもこの仕組みが暗黙的に適用されます。

## ダイジェストファイルと暗号化

ダイジェストファイルについて言及したので、これについても特徴をまとめます。

なおオブジェクト保存先ですが、prefixの設定をしない場合は、

* ダイジェストファイルは `s3://${s3-bucket-name}/AWSLogs/${aws-account-id}/CloudTrail-Digest/` 
* 証跡は `s3://${s3-bucket-name}/AWSLogs/${aws-account-id}/CloudTrail/` 

に、それぞれ生成されます。

* ダイジェストファイルはBucket先の設定いかんにかかわらず強制的にSSE-S3が適用されます。
* bucketの設定で任意のKMSを指定した上で、そのbucketをTrailの保存先にしたとしても、ダイジェストファイルは前述の通りSSE−S3で暗号化され、それはTrail側で別途KMSを指定した場合も同様。
* Trail側でログ暗号化をしない場合には、たとえS3bucketの設定で任意のCMKを指定していたとしても、ダイジェストファイルと同様SSE-S3で暗号化される
* なお当該bucketに対し、Plainにputした場合は当然bucketで設定したCMKで暗号化される。

## CloudTrailの暗号化とS3の暗号化を併用した場合

CMKをそれぞれ指定した場合には、当該S3へのPutObject、証跡およびダイジェストファイル作成において、それぞれの暗号キーは異なります。
まぁ当然といえば当然なのですが、ちょっと何が起こっているのかわかりにくいですよね。

ようは、S3のBucket暗号化の設定にかかわらず、CloudTrailにおいてはCloudTrailよるPut時の設定が、優先して適用されるということです。

以下に例を書きました。

* 設定内容
  * S3にてBucket設定としてSSE-KMS（CMK1）による暗号化を設定する
  * CloudTrailのログ設定にて暗号化を有効にし、KMS（CMK2）を設定する
  * CloudTrailのログ設定にてダイジェストファイルの生成を有効にする

* 実行結果
  * CloudTrailの証跡はCMK2で暗号化された
  * CloudTrailによるダイジェストファイルは前述の通りSSE-S3で暗号化された
  * S3のBucket設定は依然としてCMK1で設定されており、当該S3に何らかのファイルをPutしてみるとCMK1で暗号化された

CMK1とCMK2はキーが違っていればよいということを指すので、片方がKMSデフォルトサービスキーでも構いません。同じ挙動になります。

## 気をつけるべき点

CloudTrailで気をつけるべき重要な点としては、証跡の数がリージョンあたり5つまででありこれは緩和できない、ということです。
ですので、ObjectLevelLoggingは必要最小限に留めるようにつとめるのが、理想です。本当に重要な情報が保存されるBucketにのみ有効化するのがいいと思います。

## 参考

* [AWS CloudTrail データイベントで S3 バケットのオブジェクトレベルのログ記録を有効にする方法 - Amazon Simple Storage Service](https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/user-guide/enable-cloudtrail-events.html)
* [AWS CloudTrail における制限 - AWS CloudTrail](https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/WhatIsCloudTrail-Limits.html)
* [CloudTrail ダイジェストファイルの構造 - AWS CloudTrail](https://docs.aws.amazon.com/ja_jp/awscloudtrail/latest/userguide/cloudtrail-log-file-validation-digest-file-structure.html)




