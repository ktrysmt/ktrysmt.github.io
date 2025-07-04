---
layout: post
title: "cfn import をちゃんと使う"
date: 2025-06-01 09:00:00 +0900
categories: AWS
published: true
use_toc: false
---


手動のimportは手順が地味に多いのでワンショットではfixできないが、気をつけて使えばまぁまぁ便利。

### --change-set-type IMPORT の場合は普通のchange-setでリソース操作できない
よって作業単位で分割すること。


### オプションの --resource-to-import は file:/// でファイル指定できる
少しラク。

### --resource-to-import のjsonのフォーマット
例：
```
[
    {
        "ResourceType":"AWS::DynamoDB::Table",
        "LogicalResourceId":"GamesTable",
        "ResourceIdentifier": {
            "TableName":"Games"
        }
    }
]
```
* ResourceType: cfnでいうtype
* LogicalResourceId: cfn上の論理ID（手動作成したものとスタック上の名前とを紐づけるため）
* ResourceIdentifier：get-template-summaryで特定（後述）

### ResourceIdentifierはget-template-summaryで特定する
```
aws cloudformation get-template-summary --template-body file://template.yaml
```
などと打つことで対象のリソースにおいて何をResourceIdentifierに指定すればいいかわかる、のでこれを使う。

### --import-existing-resources はものによってはうまくいかない
ec2などカスタム名を持たない一部は使えないのでそういうときは諦めて手動で。
なお `change-set-type CREATE` のとき限定になる。

### --tags は同時に使えない

リソースの変更に相当する？らしく同時には使えないので外すこと。
cfn作成時はそれがわかるタグを慣習的に付けるのだが、ここだけ例外。あとでchange-setを別の理由で使うときに改めてつけること。

### 特殊なステータス

- create-change-setが通った直後: `REVIEW_IN_PROGRESS`
- execute-change-setが通った直後: `IMPORT_COMPLETE`

statusをフックに色々やるときは上記に注意。

`REVIEW_IN_PROGRESS` はcreate-change-setされただけでリソースやtemplateは空っぽの状態なので、この状態でdelete-stackをしても実害はない。安全装置的扱い。

### 流れのおさらい

1. describe, get-template-summary などで importしたいリソースを確認
2. templateを書く
3. import先のスタックをきれいな状態にしておく
4. import実行
  * type IMPORT + --resource-to-import
  * type CREATE + --import-existing-resources


