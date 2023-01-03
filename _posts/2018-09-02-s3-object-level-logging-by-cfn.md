---
layout: post
title: "S3 Object-level loggingの設定をCloudFormationで書くときのポイント"
date: 2018-09-02 03:00:00 +0900
comments: true
categories: AWS
published: true
use_toc: false
description: "CloudTrailをCFnで書くことになるんですが、ReadWriteTypeのNoneを選びたいところ実はNoneは無いのです。その対処法です。"
---


## 背景

私がCloudTrailを設定するときは、後から参照することを考慮して`Read`と`Write`、あと特定のS3Bucketの`Object-level logging`とで、TrailおよびBucketを分けて設定することが多いのです。
この`Object-level logging`をCloudFormationで書こうとしたときに、ちょっとハマったっていう話です。

## ハマったところ

下記リファレンスの通り、`Valid Values: ReadOnly | WriteOnly | All`とあり、`None`はありません。

* https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_EventSelector.html

マネコンの設定画面ではラジオボタンでNoneを選べるんですが、CFnではどうやら定義の仕方が異なるようなのです。

`Object-level logging`用と全体の監査ログ用とでBucketレベルで分かれていると管理上微妙に都合が良かったりするので、できれば`None`を選びたい...。

## 解決するコード

```yml
MyObjectLevelLoggingTrail:
  Type: "AWS::CloudTrail::Trail"
  Properties:
    EnableLogFileValidation: true
    EventSelectors:
      - DataResources:
          - Type: "AWS::S3::Object"
            Values:
              - !Sub "arn:aws:s3:::${MyTargetS3Bucket}/"
        IncludeManagementEvents: false # <= ここfalseにする
        ReadWriteType: 'All' # <= するとここの設定は無視される
    IncludeGlobalServiceEvents: false
    IsLogging: true
    IsMultiRegionTrail: false
    S3BucketName: 'my-object-level-logging-bucket'
    S3KeyPrefix: 'my-key-prefix'
    TrailName: "my-object-level-logging-trail"
```

`IncludeManagementEvents`を`false`にすれば、実態としては`None`を選んだことになります。できてよかった。
