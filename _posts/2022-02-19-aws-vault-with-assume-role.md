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

`~/.aws/credentials` は使わない。基本は以上でOKで、あとはaliasや関数で少し楽をする。

aliasの設定例；

```
alias assume1="unset AWS_VAULT; aws-vault exec account1 -s --prompt=osascript -- "
alias assume2="unset AWS_VAULT; aws-vault exec account2 -s --prompt=osascript -d 12h -- "
```

- `-d` durationはあらかじめassume先でexpire延長をしておかないと効果がないので注意。
- `--prompt=osascript` をつけるとターミナル上に平文でトークンが残らないので少し良いです。
- 毎回unsetするのだるいので`unset AWS_VAULT` を入れておく。

## 使い方

```sh
# ワンショットで
$ assume1 aws s3 ls

# セッションとして
$ assume2 zsh
$ aws s3 ls
```

## その他

クレデンシャルを平文ではなくkeychainに置けるのが最大のメリットですが、keychain timeoutがきつすぎると地味にストレスなので以下を参考に調整すると良いです。

* <https://qiita.com/minamijoyo/items/5ed3113434e51308ded1>
