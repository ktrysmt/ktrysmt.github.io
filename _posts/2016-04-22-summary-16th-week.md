---
layout: post
title: "2016 16th week 気になった記事"
date: 2016-04-22 15:09:08 +0900 
comments: true
categories: Document
published: true
---

気になった記事まとめ。

### Infrastructure as Code 再考

<http://mizzy.org/blog/2016/04/22/1/>

- プロセスステップおさらい
  - バージョン管理
  - 繰り返し可能なビルド
  - テスト
  - 継続的インテグレーション
  - 継続的デプロイ
- いわゆるデプロイなど作業の自動化が発端だが最終的にはDevOpsという観点にまで昇華され現在
- 派生するプラクティス
  - テスト駆動
    - プログラム
      - Rspec/MiniTest/xUnit
      - BDD的なものたち
  - インフラ
    - Serverspec
  - 継続的デプロイ
  - 継続的インテグレーション
- Infrastructure as Code の適応領域
  - 領域定義
    - Orchestration
    - Configuration
    - Bootstrapping
  - 領域定義がいまいちツールにマッチしてない
  - 領域をまたぐツールが多い、共通認識化されてない
  - 例えばChefは使い方によってはConfigurationとBootstrappingをまたぐし、そうなる可能性が高い
- Infrastructure as Code 関連ツール分類
  - これがわかりやすい
    - Dynamic Infrastructure Platforms
      - EC2 のような IaaS や OpenStack のような IaaS を構成するためのツール
    - Infrastructure Orchestration Tools
      - Terraform や CloudFormation のような、IaaS 上でサーバ/ネットワーク/ストレージといったリソースを制御するためのツールやサービス
      - Consul, etcd, ZooKeeper のような Configuration Registry
    - Server Configuration Tools
      - Puppet, Chef, Ansible, Itamae といったリソースの設定を行うためのツール
    - Infrastructure Services
      - プロビジョニングしたインフラを管理するためのツール
      - モニタリング、サービスディスカバリ、プロセス・ジョブ管理、ソフトウェアデプロイメントなど
  - 分類はわかりやすいが似ているツールをグルーピングしただけでどうあるべきかまでは到達しない
  - そこはやはり領域定義がはっきりすべきか

### Apline Linux 

<https://speakerdeck.com/stormcat24/qing-liang-imezishi-dai-wo-sheng-kirutamefalsealpine-linux>

- RancherOS,CoreOSとの有意差は？
  - Dockerが正式採用してる
  - AmebatvFreshで使ってるらしい
- 軽量、ISO80MB,コンテナ2MB
  - 軽さだけならRancherのが上だがどうか
- パッケージ管理
  - apk使うらしい
  - 適度に枯れてる
- リポジトリ
  - 若干追従遅い
  - ubuntuにはかなわないがubuntuは重い
- SnappyUbuntuではダメなのか
  - 正直まだ荒削りで使うの怖い感
- 結局ビルドインストール
  - まぁそれはそうなんだけどDocker使いたいって話なので...
  - 監視系やプロキシサーバはホストに入れる可能性もあるから仕方ないかも
  - http/2はnginxビルドインストールv1.9系だし

### React use-case 

<https://docs.google.com/spreadsheets/d/1aMzqpzgpI8uvBZ5fCrsIo-A8s_9-ITa1r32p04yLd3k/edit#gid=0>

- Webpack vs Browserify
- 再考の余地あり
- ほんとに半々っぽい
- React-Reduxがデファクトっぽい

### Ubuntu 16.04 Release 

<http://releases.ubuntu.com/xenial/>

- 週末触ってみよ
