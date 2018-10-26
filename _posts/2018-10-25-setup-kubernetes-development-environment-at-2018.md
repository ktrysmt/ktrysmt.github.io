---
layout: post
title: "Kubernetes開発環境の構築と周辺ツールまとめ (2018年)"
date: 2018-10-25 00:00:00 +0900
comments: true
categories: "Kubernetes"
published: true
use_toc: false
description: ""
---

今更感もありますが、備忘録もかねて最近のk8sスタックをまとめていきます。

必要なもの
----

クラスタはMacならDocker for Macを、Ubuntuならmicrok8sを使います。minikubeでもいいですが後々面倒なことになる可能性もあるので、無理に使う必要はないでしょう。

今回はMacのほうで説明します。また、仕事ではAWSをよく触るのでややAWS寄りです。

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

（がぞう）

minikubeと比べ`type: LoadBalancer`をそのまま使えるのでaddon(nginx-ingress)等の考慮が不要になるのが嬉しいです。

### 2. コンテクストスイッチ

`kubectx`でコンテクストを変更します。

```
kubectx docker-for-desktop
```

Docker for Macの設定からもコンテクストの変更が可能です。

（がぞう）

### 3. Hello World 

最後にHello worldが動くかを確認しておきましょう。

* <https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/>

必要最低限のセットアップはこれで完了です。

周辺ツールなど
----------

移行はおまけというか周辺ツールの紹介や使い方になります。なくても別に問題ないです。

### クラスタ構築

* GCPならGKE
* AzureならAKE
* AWSなら
  * EKS
  * kops
  * kube-aws

AWSならkopsかkube-awsを使います。私はkopsを使うことが多いです。

EKSならCloudFormationで構築できるので、kopsは不要です。
ただEKSの場合もワーカーノードは自前で管理していく必要があるので、いずれにしろ引き続きASGやSpotfleetなどの知識は必要になってきます。

離れ業でvirtual-kubeletを経由してfargetをワーカーノードにするテクニックもあるみたいです。
やったことはないですが。。([参考](https://aws.amazon.com/jp/blogs/opensource/aws-fargate-virtual-kubelet/))

### 構築テンプレート

* helm
* kustomize

helmかkustomize、もしくはその両方を使います。
helmは、よくメンテされている有名なOSSを手っ取り早く入れるのに最適です。
自前で社内向けにhelm chartをメンテしていくのは、、結構ツライと思います。

kustomizeはhelmほど複雑にならずとも環境ごとの設定の分離がしやすいので、導入の敷居は低いです。
ただhelmと異なり変数埋め込みなどができないので、そこについては工夫が必要になってきます。
ざっとググったかんじだとenvsubstを使う事例が散見されます([参考](https://blog.yuyat.jp/post/kubernetes-immutable-configmap/))。

### Completion

各種補完が揃っています。どれも作法が同じなので管理しやすく助かります。

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

### Aliases

エイリアスは正直好みですが、`k`や`kg`,`klo`,`kns`あたりはわりとよく使うのであるといいかもです。

私は今のところ以下をあててます。

```sh
alias k="kubectl"
alias kg="kubectl get"
alias ka="kubectl apply"
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



### Storage

* emptyDir (tmpfs)
* hostPath
* rook

hostPathはあくまで開発用途なのでなるべく使わないように実装することが望ましいと感じてます。
hostPathを使う場合は、`DirectoryOrCreate`などを指定しておくとわりと便利です([参考](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath))。

emptyDirは要するに揮発ディスクです。tmpfsを当てることも出来ます。
`emptyDir.medium`fieldに`"Memory"`を指定するとtmpfsになります([参考](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir))。

Production運用ではEBSなどをPersistensVolumeとして使う手法が知られています。
ただ、k8sでのファイルストレージをどうするかと言う問題は、まだあまり解決策を見いだせていない感じがしています。

そういう意味で、個人的に分散ストレージのRookも使ってみたいです（現状はAlpha版みたいですが）。

### Networkking

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


以上です。思い出したりなにか見つけたら追記していきます。
