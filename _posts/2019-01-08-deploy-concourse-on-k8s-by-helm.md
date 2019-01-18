---
layout: post
title: 'HelmでConcourseをデプロイしてみる'
date: 2019-01-08
comments: true
categories: Kubernetes
published: true
use_toc: true
description: 'Helmを使ってKubernetesクラスタにConcourseをデプロイする方法や、Chartに定義されている各種オプションの基礎的な部分についてまとめました。'
---

執筆時の各種バージョンは以下です。

* chart: concourse-2.0.4
* concourse: 4.2.1

また、構築においては入力補完を設定しておくと便利です（[参考][1]）。

## k8sクラスタの構築

クラスタ構築は docker-for-desktop を使うのが手っ取り早いです。

* [docker-for-desktop(Mac版)を使う例はこちら][2]

私が個人的にkopsをよく使うので、kopsの場合の例も載せておきます。

```
$ brew install kops
$ export KOPS_STATE_STORE=s3://YOUR_S3_BUCKET
$ export KOPS_CLUSTER_NAME=your-cluster-name.k8s.local
$ export ADMIN_CIDR=xxx.xxx.xxx.xxx/32
$ kops create cluster \
  --admin-access=$ADMIN_CIDR \
  --zones "ap-northeast-1c,ap-northeast-1d" --yes
```

kopsで立てる場合、`ADMIN_CIDR`にはご自身の環境のグローバルIPを指定して外部にさらされるリスクをカバーするのが手軽です。
今回は言及しませんがこの辺EKSだとIAMベース認証がマストになるため、認証はもう少しモダンで強力になります。もちろん、kopsでもロールベース・ユーザーベース認証は実現できますが、今回は特に本番運用を想定しない記事なので、複雑化を避けるため割愛し、IP制限による最低限のセキュリティ確保にとどめています。

以下はもう少しリソースを節約したい場合のオプション例です。

```
$ kops create cluster $KOPS_CLUSTER_NAME \
  --admin-access=$ADMIN_CIDR \
  --zones "ap-northeast-1c" \
  --node-size t2.medium \
  --node-volume-size 20 \
  --master-zones ap-northeast-1c \
  --master-size t2.medium \
  --master-volume-size 20 --yes
```

なお上記例ではmaster/nodeにt2.mediumなどとリソース指定しており、これは厳密にはもう少しインスタンスタイプやディスクサイズをケチることもできるのですが、クラスタそのものが不安定になってトラブったりハマっても悲しいだけなので、念のため少し余裕をもたせるために上記設定にしています。
また、上記にはありませんが `--node-count 5` というオプションで台数の変更もできます。

その他、kopsでクラスタを建てる場合の解説は以下の記事にまとめていますのでこちらも参照ください。

* [Kops on AWS コトハジメ](https://ktrysmt.github.io/blog/kops-on-aws-beginning/)

## Helmの設定

```
$ brew install kubernetes-helm
$ helm init
$ kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

## 構築してみる

### 最も単純な構築の場合

```
$ helm install --name my-release \
  --set concourse.web.kubernetes.keepNamespaces=false \
  --set concourse.web.bindPort=80 \
  --set web.service.type=LoadBalancer stable/concourse
```

最低限必要なオプションを解説します。

* concourse.web.kubernetes.keepNamespaces: `true` to `false`
  * helm deleteするときに一緒にnamespaceも消えてくれるので少し楽できます。デバッグするときは頻繁に作ったり消したりすると思うのでfalse推奨です。
* concourse.web.bindPort: 80
  * デフォルトは8080ですが80のほうがラクなので変更します。本番運用では443に変更し、証明書をあてるのがよいかと思います。
* web.service.type: loadBalancer
  * デフォルトはClusterIPのためこちらで。

引数で指定せずvalues.yamlをDLしてきて、そこに上記を書いてしまえばラクできます（後述）。

また、なれてくると `install` より `upgrade -i` のほうが便利かもしれません（当該リリースがあればUpgrade,なければInstallという挙動になる）。

それと、externalUrlを指定してあげないとログイン後リダイレクトなどがうまく動かない(デフォルトでは`127.0.0.1`へのリダイレクト)と思いますので、`helm install`後にexternalUrlを指定してupgradeしておくことをおすすめします。

kopsで建てた場合は`kubectl svc`でelbのFQDNを抜けます。それでupgradeしましょう。

```
$ ENDPOINT=$(kubectl get svc my-release-web -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
$ helm upgrade -i my-release \
  --set concourse.web.kubernetes.keepNamespaces=false \
  --set concourse.web.bindPort=80 \
  --set web.service.type=LoadBalancer \
  --set concourse.web.externalUrl=http://${ENDPOINT} stable/concourse
```

なおこの構築コマンドだとPostgresもk8sクラスタ内に作られるのでreleaseをdeleteするとDBも消えてしまうのがネックですが、あらかじめdumpしておくなど一応対策はできます。

ここまでくれば、ブラウザで http://${ENDPOINT} にアクセスし、初期設定されているローカルユーザー (ユーザー名:test / パスワード:test) でログインできるようになると思います。

### DBに外部のPostgresを使う場合

values.yamlを[githubからDL][1]して、編集して使います。

最低限の変更箇所は以下のような感じです。

（多少見やすくなるかと思い行番号も添えてます）

```
79     postgres:
80       ## The host to connect to.
81       host: <DB_HOSTNAME> # Postgresを建てているホスト名を入力
82       ## The port to connect to.
83       port: 5432
84       ## Path to a UNIX domain socket to connect to.
85       # socket:
86       ## Whether or not to use SSL.
87       sslmode: disable
88       ## Dialing timeout. (0 means wait indefinitely)
89       connectTimeout: 5m
90       ## The name of the database to use.
91       database: <DB_NAME> # 構築時のDB名を入力

(略)

699   service:
700     ## For minikube, set this to ClusterIP, elsewhere use LoadBalancer or NodePort
701     ## ref: https://kubernetes.io/docs/user-guide/services/#publishing-services---service-types
702     ##
703     type: LoadBalancer

(略)

909 postgresql:
910
911   ## Use the PostgreSQL chart dependency.
912   ## Set to false if bringing your own PostgreSQL, and set secret value postgresql-uri.
913   ##
914   enabled: false
```

docker-for-desktopまたはkopsが今回のクラスタベースなので、type は LoadBalancer でいきます。

k8sクラスタ内にpostgresはいらないので、falseを指定します。

その上で、以下のコマンドを実行します。

```
$ export CONCOURSE_USERNAME=concourse
$ export CONCOURSE_PASSWORD=password
$ helm install \
  --set web.service.loadBalancerSourceRanges\[0\]=${ADMIN_CIDR} \
  --set secrets.postgresUser=${CONCOURSE_USERNAME} \
  --set secrets.postgresPassword=${CONCOURSE_PASSWORD} \
  --values values.yaml -n my-release stable/concourse
```

kops(aws)で構築した場合はRDSを使ってPostgresを建てると思いますが、その場合はSubnetGroupやSecurityGroupを適宜設定してあげてください。

### 認証をローカル認証からGithubOAuthに変更する場合

github上のどのくくりで認証を受け付けるか、values.yamlを編集して指定します。
以下は github organization の team で指定している例です。

```
296         ## Authentication (Main Team) (GitHub)
297         github:
298           ## List of whitelisted GitHub users
299           user:
300           ## List of whitelisted GitHub orgs
301           org:
302           ## List of whitelisted GitHub teams
303           team:
304             your-organization-name:your-team-name
```

あとはGithubOAuthアプリケーションを新規に発行します。
OAuth発行時のコールバック先ですが、Concourseは `https://${HOSTNAME}/sky/issuer/callback` を期待するので、ホスト名を自身の環境のIPないしホスト名に置き換えて設定します。
そして、OAuthのID/Secretを以下のようにコマンド実行時に渡してあげればOKです。

```
$ helm install \
 --set secrets.githubClientId=${GITHUB_CLIENT_ID} \
 --set secrets.githubClientSecret=${GITHUB_CLIENT_SECRET} \
 --values values.yaml -n my-release stable/concourse
```

## その他、values.yamlで変更しておくべき項目

多数あるので、基本的なものについて抜粋します。

* concourse.worker.baggageclaim.driver: `naive` to `btrfs` 
  * btrfsも古いですが一応これが推奨のようです
* concourse.web.bindIp: 0.0.0.0
  * 絞ることもできますがLBが前段に立つのでここは開放しても問題ないです

これ以外にもいわゆるSecretsまわりに設定変更を推奨している項目がいくつかあるのですが、開発用途でトライする分にはここまでできれば一旦は困らないと思うので、割愛しています。

## おわりに

設定項目がかなり多くあるので今回は基礎的な解説にとどめましたが、それでも慣れていないとなかなか複雑に感じるかもしれません。ですので、まずは薄い設定で建ててみて、実際にいろいろ手を動かしたりしてみるのがよいと思います。

本番運用向けの細かい設定については、おって別記事にしたいと思います。

[1]: https://ktrysmt.github.io/blog/setup-kubernetes-development-environment-at-2018/#completion
[2]: https://ktrysmt.github.io/blog/setup-kubernetes-development-environment-at-2018/#1-docker-for-mac

## 参考

* <https://github.com/helm/charts/tree/master/stable/concourse>
* <https://ktrysmt.github.io/blog/setup-kubernetes-development-environment-at-2018/>

