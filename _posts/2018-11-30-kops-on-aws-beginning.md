---
layout: post
title: 'Kops on AWS コトハジメ'
date: 2018-11-30
comments: true
categories: 'Kubernetes'
published: true
use_toc: true
description: '最近kopsをよく使うようになったので、備忘録代わりに基本的な操作方法や主要なオプションの解説などをまとめました。'
---


おもにkopsを初めて触る人向けに書いています。

kopsにはオプションが多くあるのですが、細かい設定についてはProductionReadyな設定集として、別記事にまとめようかと思います。

## インストール

Macならbrewで入ります。本記事では1.10.0をベースにすすめます。

```
$ brew install kops
$ kops version
Version 1.10.0
```

入力補完を設定しておくと便利です([参考](/blog/setup-kubernetes-development-environment-at-2018/#completion))。

```
if (which kops > /dev/null); then
  function kops() {
    if ! type __start_kops >/dev/null 2>&1; then
        source <(command kops completion zsh)
    fi
    command kops "$@"
  }
fi
```

## 事前に必要なもの

### S3 bucket

kopsはterraformのようにstateの保存先としてs3を使い、常時状態を保存していくようです。

S3Bucketを作成しておきます。

Stateの情報は重要度が高いので、念の為暗号化をしておくのをおすすめします。

```
$ aws s3api create-bucket --bucket <YOUR_BUCKET_NAME>
$ aws s3api put-bucket-encryption --bucket <YOUR_BUCKET_NAME> --server-side-encryption-configuration '{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }
  ]
}'
```


### IAM

state保存用のS3Bucket作成権限とは別に、クラスタを構築するためのIAM権限を用意します。
S3Bucketの作成も含め全てフル権限でやってもいいっちゃいいのですが、可能な限り構築に必要な最小限の権限でやるほうが精神衛生上よいかと思います（特に本番運用を視野にいれる場合は）。
現場によってはS3やIAMまわりは下手にフル権限を渡すと困ることがあると思うので、なるべく制限してあげるといいと思います。

ポリシー例は以下です。

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:GetContextKeysForCustomPolicy",
                "iam:GetAccountPasswordPolicy",
                "iam:DeleteAccountPasswordPolicy",
                "iam:SimulateCustomPolicy",
                "iam:UpdateAccountPasswordPolicy"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:*InstanceProfile*",
                "iam:*Role*",
                "iam:*Policy*"
            ],
            "Resource": [
                "arn:aws:iam::*:policy/*",
                "arn:aws:iam::*:instance-profile/*",
                "arn:aws:iam::*:role/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:HeadBucket"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::<YOUR_BUCKET_NAME>",  // さっきつくったS3のBucket名を入れる
                "arn:aws:s3:::<YOUR_BUCKET_NAME>/*"
            ]
        }
    ]
}
```

それでもroleやpolicy,instance-profileへの権限はどうしても必要になってしまうのですが。。

なお今回は`.k8s.local`(Gossipプロトコル)を使うので、Route53関係の権限は含まれていません(後述)。


### 環境変数

kopsには自動で読み込まれる環境変数があります。これを事前に設定しておくと手間が減ってラクできます。
設定しておきましょう。

```
$ export KOPS_STATE_STORE=s3://<YOUR_BUCKET_NAME>
$ export KOPS_CLUSTER_NAME=<YOUR_NAME>.k8s.local
```

$KOPS_STATE_STOREと$KOPS_CLUSTER_NAMEは、定義しておいてあげるとkopsコマンドを打つ際いちいち--stateや--nameで指定する手間が省けるのでおすすめです。

$KOPS_CLUSTER_NAMEですが、名付けをするときに`.k8s.local`で終わるようにします。今回は名前解決にkopsが提供する[Gossipプロトコル](https://ja.wikipedia.org/wiki/%E3%82%B4%E3%82%B7%E3%83%83%E3%83%97%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB)を使うためです。
kopsはクラスタ名が`.k8s.local`で終わる場合、自動的にKubernetesクラスタ内のリソースを使用してGossipプロトコルベースに名前解決するように構築してくれます。

パフォーマンスや安定性・速度面などでRoute53に依存させるほうがよい場合が多いのですが、Route53権限は体感的にやや強い気がして気がひけるので、開発用途等で構築することも踏まえ、今回はGossipで構築します。





## クラスタを作る

kops create cluster を主に使っていきます。

kops create cluster の流れとしては、以下のような2段階の流れをとります。

1. S3にkopsの設定情報を保存
2. それらをもとにAWSの実リソースを作成

kops create cluster を打つと、オプションで指定しない限りはこの`1.`と`2.`を自動的に行います。

また、kopsが生成する主な論理リソースとしては cluster, instance-group, secretがあります。

clusterは主にクラスタ構築のための設定情報で、instance-groupはAWSでいうところのAutoScalingGroupの設定に近い情報、secretは鍵などのクレデンシャル情報を指します。

これらを踏まえて、クラスタをつくっていきます。

### manifestファイルを生成する

前述の通り kops create cluster は自動でクラスタを構築してくれますが、設定ファイルはそのままでは手元に残りません。
バージョン管理していきたい都合上、手元にどういう設定なのかがわかるファイルを残したくなります。
これは kops create cluster のオプションで解決できます。

なおkopsの設定情報についてですが、これはk8sライクなmanifestファイルとして定義されています。

以下は、クラスタ構築のためのmanifestを生成する最もシンプルなコマンドの例です。

```
$ kops create cluster --name $KOPS_CLUSTER_NAME \
  --dry-run=true \
  --zones="ap-northeast-1a,ap-northeast-1c,ap-northeast-1d" \
  --master-zones="ap-northeast-1a" \
  --output=yaml > manifest.yaml;
```

このように`--dry-run=true`と`--output`を両方使うことで、構築やs3への設定保存はせずに manifestだけを吐き出すのを実現できます。

manifestには cluster と instance-group の情報が記述されます。

### manifestからクラスタを構築する

生成したmanifestを使ってclusterを構築するには-fオプションを使います。

```
$ kops create -f manifest.yaml
```

kops create cluster ではない点に注意です。

なお、kops create -f をした時点では、まだ`1. S3にkopsの設定情報を保存`がされただけであり、VPCやEC2などの実リソースは作成されない点にも注意です。

S3にPutされた情報をもとにリソースを配備していくのは以下のコマンドです。

```
kops update cluster --yes
```

--yesオプションを外すといわゆるdryrunとなり、具体的に生成されるリソースについての説明が標準出力に出ます。

### クラスタ構築後のチェック

構築にあたっては実際にEC2やELBが作られたり、ELBのTargetGroupのInServiceを待ったりするため、しばらく時間がかかります。

kops validate cluster で正常に処理されているかの状態をチェックできます。

## クラスタの設定を変更する

前項で、manifestベースでの設定変更のやり方を書きました。
設定変更も、manifestベースに行っていきましょう。

kops replace -fをつかいます。

```
kops replace -f manifest.yaml
```

create -f とおなじくこれだけではS3のStateが更新されただけなので、反映するために、

```
kops update cluster --yes
```

を打ちます。

なお、内部で動いているkubernetesの設定を変更する場合や、kopsが動かすEC2のInstanceTypeを変更したりディスクサイズを変更したりした場合は、

kops update clusterだけでなく、kops rolling-update clusterが必要になります。

rolling-updateが必要かどうかは kops rolling-update cluster と打つと確認できます。
これも--yesオプションがあり、実際にrolling-updateを適用する場合は　kops rolling-update cluster --yes と打ちます。

## クラスタを削除する

最後に削除しておきましょう。

```
kops delete cluster --yes
```

これも同じく--yesが必要で、外せばdry-runになります。
事前に消えるリソースの確認ができるのでdelete時は一度dry-runするのを推奨します。
万が一$KOPS_CLUSTER_NAMEを間違えていたりすると、事故につながってしまうので。

## その他基本的なオプションについて

### InstanceTypeを変えたい

開発用途であれば小さい安いインスタンスで一旦済ませたい事が多いと思います。
その場合はmasterについては`--master-size="t2.micro"`、
workerについては`--node-size="t2.micro"`というふうに指定します。

### ディスクサイズを変えたい

EBSのディスクサイズについては`--node-volume-size=20`、`--master-volume-size=20` という感じです。

ただあまりにサイズが小さいとスペック不足気味になり(特にmaster)処理が遅れるかなにがしかエラーを吐くことも考えられるので、
t2.smallやt2.mediumくらいにしておくといいかもしれません。

また、ディスクサイズについてもmasterはetcdのおかげでディスクI/Oが激しくサイズもk8sアプリケーションの配置数によっては大きくなってくるので、
あまり小さくしすぎないほうがいいかもしれないです。

### 起動時の台数を変えたい

masterは`--master-count=1`、workerは`--node-count=1`という感じにオプションでAutoScalingでの台数を指定できます。

### apiにSecurityGroupを当てたい

デフォルトで立つELBはSGのInboundが`0.0.0.0/0`と全開放になっており気になると思います。
これは`--admin-access`というオプションでSecurityGroupを当てることで対処可能で、`--admin-access="x.x.x.x/32"`とCIDRで指定できます。

おもちゃにされても困るので設定しておくといいと思います。

## コマンドのおさらい

```
# 環境変数
$ export KOPS_STATE_STORE=s3://<YOUR_BUCKET_NAME>
$ export KOPS_CLUSTER_NAME=<YOUR_NAME>.k8s.local

# manifest生成
$ kops create cluster --name $KOPS_CLUSTER_NAME \
  --dry-run=true \
  --zones="ap-northeast-1a,ap-northeast-1c,ap-northeast-1d" \
  --master-zones="ap-northeast-1a" \
  --output=yaml > manifest.yaml;

# manifestをs3へput
$ kops create -f manifest.yaml

# リソース反映
$ kops update cluster --yes

# ローリングアップデート
$ kops rolling-update cluster --yes

# manifestを編集したので再反映
$ kops replace -f manifest.yaml
$ kops update cluster --yes
```

## リファレンス

* <https://github.com/kubernetes/kops/>
* <https://github.com/kubernetes/kops/blob/master/docs/aws.md>
* <https://ktrysmt.github.io/blog/setup-kubernetes-development-environment-at-2018/>

