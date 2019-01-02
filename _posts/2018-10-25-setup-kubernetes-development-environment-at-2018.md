---
layout: post
title: "Kubernetes開発環境構築と周辺ツールまとめ (2018年)"
date: 2018-10-25 00:00:00 +0900
comments: true
categories: Kubernetes
published: true
use_toc: true
description: ""
---

今更感もありますが、備忘録もかねて最近のk8s開発環境まわりをまとめていきます。

必要なもの
----

クラスタはMacならDocker for Macを、Ubuntuならmicrok8sを使います。minikubeでもいいですが後々面倒なことになる可能性もあるので（あったので）、無理に使う必要はないかなと。

今回はMacのほうで説明します。


* Docker for Mac 18.06
* kubectl 1.12.0
* kubectx 0.6.1

インストール
------

それぞれbrewで入ります。

```
brew cask install docker
brew install kubernetes-cli
brew install kubectx
```

セットアップ
----------

### 1. Docker for Mac

設定画面からKubernetesを有効化しましょう。
マシンリソースは気になるほどではないですがそれなりに食うので、適宜On/Offするのもよいかと。

![](/assets/images/blog/setup-kubernetes-development-environment-at-2018/docker-for-mac.png)

minikubeと比べ`type: LoadBalancer`をそのまま使えるのでaddon(nginx-ingress)等の考慮が不要になるのが嬉しいです。

### 2. コンテクストスイッチ

`kubectx`でコンテクストを変更します。

```
kubectx docker-for-desktop
```

Docker for Macの設定からもコンテクストの変更が可能です。

![](/assets/images/blog/setup-kubernetes-development-environment-at-2018/k8s-context.png)

### 3. Hello World

最後にHello worldが動くかを確認しておきましょう。

* <https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/>

必要最低限のセットアップはこれで完了です。

周辺ツールなど
----------

以降はおまけというか周辺ツールの紹介や使い方になります。なくても別に問題ないです。

### クラスタ構築

IaaSごとにマネージドがあるので

* GCPならGKE
* AzureならAKE
* AWSならEKS

また、EKSはまだ東京リージョンにまだ来ていないので、
* kops
* kube-aws

AWSならkopsかkube-awsを使うことになると思います。私は仕事ではAWSがメインでして、構築には主にkopsを使ってます。

EKSならCloudFormationで構築できるので、kopsは不要です。
ただEKSの場合もワーカーノードは自前で管理していく必要があるので、いずれにしろ引き続きASGやSpotInstanceなどの知識は必要になってきます。

離れ業でvirtual-kubeletを経由してfargetをワーカーノードにするテクニックもあるみたいです。
やったことはないですが。。([参考](https://aws.amazon.com/jp/blogs/opensource/aws-fargate-virtual-kubelet/))

### 構築テンプレート

* helm
* kustomize

helmかkustomize、もしくはその両方を使います。
helmは、よくメンテされている有名なOSSを手っ取り早く入れるのに最適です。
自前で社内向けにhelm chartをメンテしていくのもありかと思います。

kustomizeはhelmほど複雑にならずとも環境ごとの設定の分離がしやすいので、導入の敷居は低そうです。
ただhelmと異なり変数埋め込みなどができないので、そこについては工夫が必要になってきます。
ざっとググったかんじだとenvsubstを使う事例が散見されます([参考](https://blog.yuyat.jp/post/kubernetes-immutable-configmap/))。

### Completion

各種補完が揃ってます。どれも作法が同じなので管理しやすく助かります。

k8s系の補完はやや重いことがあるので、簡易的なlazyloadを書いています([参考](https://github.com/kubernetes/kubernetes/issues/59078#issuecomment-363384825))。

```sh
# kubectl
if (which kubectl > /dev/null); then
  function kubectl() {
    if ! type __start_kubectl >/dev/null 2>&1; then
        source <(command kubectl completion zsh)
    fi
    command kubectl "$@"
  }
fi

# kops
if (which kops > /dev/null); then
  function kops() {
    if ! type __start_kops >/dev/null 2>&1; then
        source <(command kops completion zsh)
    fi
    command kops "$@"
  }
fi

# helm
if (which helm > /dev/null); then
  function helm() {
    if ! type __start_helm >/dev/null 2>&1; then
        source <(command helm completion zsh)
    fi
    command helm "$@"
  }
fi
```

上記の例は、zshのcompletionを呼んでますが、bashもあります。`kubectl completion bash` でOKです。

また、REPLのように補完が効く`kube-prompt`もあります。
あまり凝った設定をしていない場合はこちらのほうが手っ取り早く体感できてよさそうです。

- <https://github.com/c-bata/kube-prompt>

![](https://github.com/c-bata/assets/raw/master/kube-prompt/kube-prompt.gif)

ほかのコマンドもいろいろ組み合わせる関係で普段はあまり使わないんですが、集中してオペレーションするときにはよさそうです。

### Aliases

エイリアスは正直好みですが、`k`,`kg`,`klo`,`kex`あたりはわりとよく使うのであるといいかもです。

私は今のところ以下をあててます。

```sh
alias k="kubectl"
alias kg="kubectl get"
alias kgp="kubectl get pods"
alias ka="kubectl apply -f"
alias kd="kubectl describe"
alias krm="kubectl delete"
alias klo="kubectl logs -f"
alias kex="kubectl exec -i -t"
alias kns="kubens"
alias kctx="kubectx"
```

[エイリアスをまとめたリポジトリ](https://github.com/ahmetb/kubectl-aliases)なんかもあります。

### Tmux, Prompt

contextやnamespaceをプロンプトやtmuxステータスに表示しておくと便利です。

kubectlコマンドから抜き出して自分で書いてもいいですが、リポジトリもあるのでこれらを使うのもいいかと。

* https://github.com/jonmosco/kube-ps1
* https://github.com/jonmosco/kube-tmux

私はtmuxのステータスを自前で書いている関係で、以下の要領でstdoutを抜き出してそれぞれ使っています。

```sh
local kubens=$(kubectl config view --minify --output 'jsonpath={.contexts..context..namespace}' 2>/dev/null)
local kubectx=$(kubectl config current-context 2>/dev/null)
```



### リソース監視

<https://github.com/astefanutti/kubebox>
* kubeboxはCUIベースのリソースモニタリング。htopのk8s版といった趣。htps://kube.shというウェブインターフェイスもあるのが面白い。

<https://github.com/hjacobs/kube-ops-view>
* リソースの使用率やpodsの配置状況を可視化してくれる。

<https://github.com/bitnami-labs/kubewatch>
* 監視リソースを指定してSlackに通知してくれる。helmで入れる。

<https://appscode.com/products/searchlight/>
  * kubernetes-dashboardのようなリッチなGUI。アラート通知にも対応してるぽい。
  * appscodeからはk8s向けのツールがほかにも多数リリースされている様子。HAProxyをk8s上に配置するVoyager, k8s上にPostgresなどのRDBMSを配置するKubeDBなど。


### Storage

* emptyDir (tmpfs)
* hostPath
* Rook

hostPathはあくまで開発用途なのでなるべく使わないように実装することが望ましいと感じてます。
使う場合は`DirectoryOrCreate`が便利です([参考](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath))。

emptyDirは要するに揮発ディスクです。tmpfsを当てることも出来ます。
`emptyDir.medium`fieldに`"Memory"`を指定するとtmpfsになります([参考](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir))。

Production運用ではEBSなどをPersistensVolumeとして使う手法が知られています。
ただ、k8sでのファイルストレージをどうするかと言う問題は、まだあまり解決策を見いだせていない感じがしています。

そういう意味で、個人的に分散ストレージのRookも使ってみたいです（現状はAlpha版みたいですが）。

### Networking

アーキテクチャには大まかに L7overlay, L3/L4, BPF とあり、それぞれの主要なライブラリとしては、

* L7 overlay: Weave
* L3/L4: Calico
* BPF: Cilium

という感じですが、ほかにもいろいろな実装があります([参考](https://kubernetes.io/docs/concepts/cluster-administration/networking/))。
Overlay方式についてはFlannel、Romanaなどもよく各所で言及されています。
いわゆるSecurityGroupなどのネットワークレベルでの通信制御が要件に入ってくることはもはやアタリマエレベルだなと感じており、そうでなくてもMicroservicesなアーキテクチャにある場合にはほぼ確実に必要になってくる部分です。開発段階ではじめから導入しておくといいと思います。

### 権限まわり

* kiam
* kube2iam

Annotationの操作でpods毎にIAMを付与できるようになるので、実用的です。
kube2iamよりkiamのほうが安定性はよいようです。
開発中は特に気にしなくてもいいかもしれませんが、各種マネージドサービスとの連携などの結合テストの段階になると必要になってくると思います。


以上です。

~~思い出したりなにか見つけたら追記していきます。~~

- 2018/10/29 リソース監視の項目を追加しました。
- 2018/10/31 kube-promptを忘れてたのでCompletionに追加しました。


