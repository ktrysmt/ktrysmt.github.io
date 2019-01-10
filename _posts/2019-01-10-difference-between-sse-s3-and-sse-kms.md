---
layout: post
title: 'SSE-S3とSSE-KMSの違いを知る'
date: 2019-01-10
comments: true
categories: AWS
published: true
use_toc: true
description: 'KMSは大変便利で私の好きなAWSマネージドサービスのひとつなのですが、初見ではSSE-S3とSSE-KMSの違いはかなりわかりにくいかなと思います。そこで今回はS3のサーバーサイド暗号化においてよく使われるSSE-S3とSSE-KMSの違いについてまとめました。'
---

結論言ってしまうとSSE-S3の場合は鍵を利用したかどうかの監査証跡が取れないことがネックになります。一方でSSE-S3は追加料金が発生せず、APIコールの制限を考慮する必要もないのがメリットです。


## SSE-KMSの特徴
* キーポリシーでCMK(カスタマーマスターキー)への個別のアクセス制御ができる
* KMSに対するAPIコールに当たるため、監査証跡(CloudTrail)を残すことが可能
* 暗号化キーの作成および管理ができる
* 当該AWSアカウントのリージョンごとに一意に作成されるデフォルトサービスキーを使用することもできる
* KMS利用分の[追加料金][2]が発生
* S3に対するPut/Getが非常に多くなる場合には KMS の [APIコールの制限][1] の考慮が必要

## SSE-S3の特徴
* キーに対する個別アクセス制限は不可、実質的なアクセス制限は署名バージョン4による当該AWSアカウントID以外の拒否のみとなる
* 監査証跡(CloudTrail)には残らない
* KMSに比べキーの選択肢は無く提供されるもののみ使用できる
* SSE-S3のための追加料金は発生しない
* APIコールの制限に関して考慮は不要

## 所感

SSE-KMSはCloudTrailで証跡が取れるので、暗号化されたか・複合されたかどうかについての証跡を取っておきたいObjectがある場合には、SSE-KMSがいいと思います。
クレデンシャルや鍵、マスターデータなどの重要な情報が保存されるBucketが向いていそうです。
一方、KMSのコール制限は緩いほうですがそれでも単位時間あたりに大量にコールすることが想定される場合には注意が必要かなと思います。
KMSに限りませんが、AWSのサービス制限は緩和できないものも多数あるため、あらかじめAWSアカウントを用途ごとに分けてシステム設計をしていくと失敗しにくいと思います。

SSE-S3はAPIコール制限や料金などを気にする必要がないので、大量に自動生成されるファイルのうち比較的重要な情報を暗号化しておきたい場合に有効かなと思います。
筆頭はログファイルかなと思います。IPアドレスなど個人情報に当たる可能性がある情報が含まれる場合など、漏洩や内部破壊行為などに対する防御効果が期待できます。
SSE-S3の固定のキーポリシーはSSE-KMS同様署名バージョン4で保護されるので、第三のAWSアカウントから復号しようとしても署名の不一致で復号に失敗するためです。

## 参考

* [制限 - AWS Key Management Service][1]
* [料金 - AWS Key Management Service][2]
* [Amazon Simple Storage Service (Amazon S3) で AWS KMS を使用する方法 - AWS Key Management Service](https://docs.aws.amazon.com/ja_jp/kms/latest/developerguide/services-s3.html)

[1]: https://docs.aws.amazon.com/ja_jp/kms/latest/developerguide/limits.html
[2]: https://aws.amazon.com/jp/kms/pricing/
