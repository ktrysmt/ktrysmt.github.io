---
layout: post
title: "S3の Object-level logging の設定をCloudFormationで書くときのポイント"
date: 2018-09-02 03:00:00 +0900
comments: true
categories: ""
published: true
use_toc: false
description: "CloudTrailをCFnで書くことになるんですが、ReadWriteTypeのNoneを選びたいところ実はNoneは無いのです。その対処法です。"
---


## 背景

CloudTrailを設定するときは、後から参照することを考慮して`Read`と`Write`、あと特定のS3Bucketの Object-level logging とで、Trailおよびログ用Bucketを分けて設定することが多いです。
この Object-level logging を CloudFormation で書くときに、ちょっとハマった話です。

## ハマったところ

下記リファレンスの通り、`Valid Values: ReadOnly | WriteOnly | All`とあり、`None`はありません。

* https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_EventSelector.html

マネコンの設定画面ではラジオボタンでNoneを選べるんですが、CFnではどうやら定義の仕方が異なるようなのです。

## 解決したコード

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

`IncludeManagementEvents`を`false`にすれば、実態としては`None`を選んだことになります。
