---
layout: post
title: "aws-vaultでassume-roleする"
date: 2022-02-19 00:05:00 +0900
comments: true
categories: AWS
published: true
use_toc: false
description: ""
---

かなり今更感ありますがどうなってると一番ラクかってのが地味にわかりにくいので自分用にまとめます。

## セットアップ

```
$ brew install aws-vault
$ aws-vault add default
Enter Access Key Id: XXXXXXXXXXXXXXXXXXXX
Enter Secret Key: YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY

$ cat <<EOF> ~/.aws/config
[default]
mfa_serial = arn:aws:iam::000000000000:mfa/username
cli_pager =
region = ap-northeast-1

[profile account1]
role_arn = arn:aws:iam::111111111111:role/rolename
source_profile = default

[profile account2]
role_arn = arn:aws:iam::222222222222:role/rolename
source_profile = default

(以下略)

EOF
```

基本はこれでOKで、あとはaliasや関数で工夫。
今回はあんまり意味がないかもだけど zsh なら global alias で設定しておくと展開して修正して実行したりとちょっと便利にできます([参考](https://blog.patshead.com/2012/11/automatically-expaning-zsh-global-aliases---simplified.html))。

```
alias -g awsvault_1="unset AWS_VAULT; aws-vault exec account1 --prompt=osascript -- "
alias -g awsvault_2="unset AWS_VAULT; aws-vault exec account2 --prompt=osascript -d 12h -- "
```

手癖的におなじセッションで作業することが多くて `unset AWS_VAULT` も入れてしまってます。
`-d` durationはassume先でもexpireの延長をしないと無意味なので設定を忘れないように。
`--prompt=osascript` をつけるとターミナルに平文でトークンが残らないので少し良いです（気持ちの問題）。

## 使い方

```sh
# ワンショットで
$ awsvault_1 aws s3 ls

# セッションとして
$ awsvault_2 zsh
$ aws s3 ls
$ aws iam list-groups # ...以下略
```

## おわりに

SSOないしSAMLでというのも組織によってはあるとは思いますが、個人的にはことAWS（というかIaaS）に関してはあえて独立して管理していたほうがリスク分散になると考えてます。
コーポレートとプロダクト（サービス）は晒されているリスクの種類とコンテキストが異なるため。

平文で置かないとか IAM User 直では使わないとかなんか色々説明すっ飛ばしてますが、ひとまずこれだけでセットアップは完了です。便利な世の中に感謝。
