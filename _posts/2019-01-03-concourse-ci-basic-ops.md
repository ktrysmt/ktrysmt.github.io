---
layout: post
title: 'Concourseの基本操作まとめ'
date: 2019-01-03
comments: true
categories: CI/CD
published: true
use_toc: false
description: 'お仕事でConcourseCIを建てたり運用したりすることが多いのですが、先日改めてこれを解説する機会があり、基本的な操作方法を軽く解説を添えつつまとめてみました。'
---

構築方法や運用ノウハウみたいなものは今回は書いていません、別記事で上げたいと思います。

使用するバージョンは`4.2.1`を想定しています。

## 1. Download fly command 

Concourseのオペレーションを行うためのコマンドが`fly`です。
これは建っているConcourseサーバーのGUIからDLできるので、取得しておきましょう。
ちなみに[公式サイトからもDLできます][1]。

[1]: https://concourse-ci.org/download.html

建っているサーバーのバージョンと揃える必要があるのでそこだけ注意ですが、
バージョンを同期させるサブコマンドがあるので、それを使うとラクです。

## 2. Basic operations

`fly help`で出てくる各種サブコマンドのうち、よく使うもの・基本的なものを解説します。
なお各サブコマンドには1文字〜2文字のaliasが提供されているので、慣れてきたらaliasを積極的に使うといいと思います。

### ログイン、環境

基本的には`$ fly -t main login -c <URL>`です。
localuserが提供されている場合は`-u`と`-p`で直接UserID/Passwordを入力することでもログインできます。
GithubOAuthなどのOAuth系のログインが提供されている場合には sky token を使ったログインになります。

なお、以降targetはmainという設定で、解説を進めます。

```
# login (by local user account)
$ fly -t main login -u test -p test

# login (by sky token)
$ fly -t main login -c <URL>
```

ログイン状態やログイン先環境の確認は以下です。
組織内で複数Concourseを建てている場合は target 間違いに注意。

```
# login status
$ fly -t main status
$ fly -t main targets
```

### バージョン番号の同期

バージョン番号が一致していない場合オペレーションがうまくいかない（コマンドがfailする）可能性があるので必ず一致させておきましょう。
これを打つだけで同期してくれるので、便利です。

```
# sync command version
$ fly -t main sync
```

### パイプライン処理を書く、セットアップする

Concourseの基本の流れは、パイプラインを書く→チェックする→上げる→unpauseする→キックする、です。
といっても`validate`はかなり簡易的なチェックしかしてくれないですが、無いよりはマシかなと。

Concourseにはリポジトリ等の定期的なチェック（ポーリング）の仕組みが備わっていることから、`set-pipeline`をしただけでは実行開始とならず、上げたあとにpause状態を解除する`unpause`が必要になります。

また、Pipelineを実行するにあたっての実行の単位はジョブ単位となるので、`trigger-job`をする場合はジョブ名に注意です。といっても、だいたいの場合パイプラインの先頭に登録されているジョブをキックするはずです。

パイプライン処理の中で中盤のジョブをキックするようなケースは、承認フローやレビューフローを設けるような場合があげられます。

```
# validate
$ fly -t main validate-pipeline -c helloworld/pipeline.yaml

# set pipeline
$ fly -t main set-pipeline -c helloworld/pipeline.yaml -p example-helloworld

# unpause it
$ fly -t main unpause-pipeline -p example-helloworld

# kick the job
$ fly -t main trigger-job -j example-helloworld/job-hello-world
```

### ジョブの状態を確認する

基本は`builds`で全部見れます。`-p`や`-j`でフィルタできます。
通常はGUIから確認すると思うのであまり使いどころがないように思われるかもしれませんが、BotkitやHubotなどを経由してChatOpsする場合に使います。
`wait`も同様で、ChatOps向けコマンドの実装や他のCIとの連携のときに便利なコマンドです。

```
# list builds
$ fly -t main builds
$ fly -t main builds -p example-helloworld
$ fly -t main builds -j example-helloworld/job-hello-world

# wait the job
$ fly -t main wait -j example-helloworld/job-hello-world
```

## 3. Enhanced operations (maintenance)

主にConcourseのお守りをするときに使うコマンドです。
重要なのは`prune-worker`です。ConcourseのWorkerはジョブキックのたびにスポーンするのではなく基本的に常駐するため、プロセスのゾンビ状態のように応答しなくなってしまうことがたまにあります（ありました）。少しずつ安定してきて入るようなのですがこのWorkerの不安定感は私が使い始めた2系のころから（だいぶマシにはなりましたが）あまり変わっていないというのが実感です。（TCPコネクションを張りっぱなししているっぽく、それが切れると発生するように見える）

そういった経緯のなかで一時しのぎ的にだとは思うのですがWorkerをクリーンアップしてくれる`prune-worker`コマンドが生まれました。

WorkerについてはほかにWorkerを追加する`land-worker`、Workerを退役する`retire-worker`がありますが、通常はサーバー等のスペックのためにWoeker数は一定数決まっていると思いますので、あまり使わないかなと思います。Worker数を増やすようなときは設定を変えて再起動したりプロビジョニングをし直したりクラスタの設定を変えたりなど、すると思うので。

あと`abort-build`ですが緊急停止ボタン的に使うことがあるかもしれないので、一応覚えておくといいかもです。

```
# analytics
$ fly -t main workers
$ fly -t main volumes
$ fly -t main containers

# clean up worker status
$ fly -t main prune-worker

# stop build
$ fly -t main abort-build -b <build_id>
```

## 4. enhanced operation (hijack the container)

最後に`hijack`コマンドを紹介します。

これはいわゆる`docker exec`や`kubectl exec`のことで、実行中のジョブコンテナの中に入って、ステータスやプロセス、ファイルの状態などを on the fly に確認できるコマンドです。
普段は使わないかなと思いますが、なんらかの理由でジョブがスタックしてしまった場合に非常に有用です。
これに関連したデバッグテクニックとして、調子の悪いジョブに対し、当該ジョブのタスク内でわざと長時間waitさせたり無限ループさせることで`hijack`できるように時間を稼ぎ、中身を確認しにいく...というやり方があります。Dockerfileデバッグとほぼ一緒ですが、Concourseもコンテナベースでのタスク実行という構成ですから、同じやり方が通用します。

```
# hijack container task
$ fly -t mian hijack -j <pipeline/job>
```

なお参考までに、以下の一連のコマンドは上記の話を踏まえたもので、わざとsleepさせるタスクを書いて、当該タスクコンテナにhijackするという流れを体験できます。

```
# set sleep pipeline
$ fly -t main set-pipeline -c sleep/pipeline.yaml -p example-sleep

# kick the job
$ fly -t main trigger-job -j example-sleep/job-sleep

# confirm job stacking...
$ fly -t main builds -j example-sleep/job-sleep

# or, watch it 
$ watch -n 1 fly -t main bs -j example-sleep/job-sleep

# hijack it
$ fly -t main hijack -j example-sleep/job-sleep sh
```

ちなみに`sleep/pipeline.yaml`の中身は以下です。

```yaml
---
jobs:
- name: job-sleep
  public: true
  plan:
  - task: call-sleep-task
    config:
      platform: linux
      image_resource:
        type: docker-image
        source: {repository: busybox}
      run:
        path: sh
        args:
          - -exc
          - |
            echo "in sleep 1000sec..."
            sleep 1000
```


## 参考

- <https://concourse-ci.org/>