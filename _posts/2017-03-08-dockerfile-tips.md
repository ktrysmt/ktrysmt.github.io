---
layout: post
title: 今更まとめるdockerコンテナの開発Tips
date: 2017-03-08 09:00:00 +0900
categories: Docker
published: true
use_toc: false
---

最近ようやっと本格的に触りだしたので備忘録的に書きます。

プラクティスやアンチパターン的な話なので、
いわゆるチートシートやチュートリアルについては他記事を参照ください。

## Dockerfile関連

### `FROM`は慎重に選ぶ

1. なるべく公式のDockerfileを使う。
  * refs: <https://hub.docker.com/explore/>
  * だいたい必要なものは揃ってるはず
2. ユーザーコンテナは自分でDockerfileを作成する時の参考に
  * Download数の多いDockerfileは特に参考になる
3. 軽量コンテナについて
  * はじめはなるべく標準のコンテナを使い、軽量コンテナの使用は一旦避ける
  * まずはコンテナ運用のメリット・デメリットを体感し自分が求めるものかどうかを見極める
  * 軽量コンテナ自体は`node:7-slim`,`golang:alpine`など公式的に提供されているので使う場合はこれらを使う
  * 軽量コンテナの検討や導入は、既にDockerを運用していたり大量配備にまつわる悩みを抱える段になってからでも遅くはない
  * 依存関係のトラブルはハマると時間食い虫なので注意
     * 例えば3rd Partyのライブラリを使用したいときなど

### Dockerfileのお作法

1. とりあえず公式の[BestPractices](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)を一読。
  * [日本語版](http://docs.docker.jp/engine/articles/dockerfile_best-practice.html)もあります
2. `ENTRYPOINT`をベースコマンドとして使い、デフォルト引数として`CMD`を使う
  * 適当にググるだけだと`CMD`使う例が多くヒットするので混乱する
  * 要するに`sh -c`が`ENTRYPOINT`の初期値で、その引数が`CMD`という位置づけ、`ENTRYPOINT`ナシで`CMD`だけ書いても動くのはこれが理由
3. `ADD`はリモートからのリソース取得ができる、`curl`,`wget`の代わりだと思えばよい
  * `ADD`には他にもtarによるローカル圧縮ファイルの自動解凍の仕組みが備わってる
  * 便利っちゃ便利だが、わからず使ってると混乱するかも
4. `COPY`は単純なローカルからコンテナへのファイル転送として使う
  * `ADD`と住み分ける
5. `RUN cd && command`ではなく`WORKDIR`を使う
6. ホストボリュームのマウントをコマンドラインから指定するかDockerfile内の`VOLUME`で指定するかはケースバイケース
  * まだあまり使い込んでいないので、もうちょっと使い慣れたらUpdateしたい
  * [ストレージドライバ](https://goo.gl/9S8AS4)まわりの話から察するに結局NFS運用がつらい的な問題にぶつかりそうな気配

### ディレクトリ構成

レイヤーを考慮した結果、だいたい以下のような構成になった。

```
.
├── base
│   └── Dockerfile
├── middleware
│   ├── Dockerfile
│   └── middleware.conf
├── app
│   └── Dockerfile
└── README.md
```

* 各Dockerfileごとに必要な設定ファイルは当該ディレクトリ内に同居させる
  * Nginxのconfやfluentdのconfなど
  * MySQLコンテナ内の`/docker-entrypoint-initdb.d`に設置したいfixture.sqlなど

### レイヤーを効果的に使う
1. レイヤーを細かく切るほうがビルド待ち時間を減らせる
  * 既にAnsibleやChefの資産があるのなら`role`の概念をそのままレイヤーに置き換えて考えれば悩みが減る
  * 変更余地の無さそうなコマンドは早めに`base`レイヤーや`packages`,`middleware`レイヤーなどの名前をつけて切り分ける
  * レイヤーを活用することでビルドの待ち時間を削減でき、効率的な編集作業が可能になり、原因の切り分けもしやすくなる。
2. `USER`をDockerfile内で何度も切り替えるのはどちらかというと悪手
  * でも`sudo`を使うよりはマシ
  * はじめに`root`で依存をまとめて入れておき、以後のレイヤーで`USER`を切り替えて`RUN`,`EXPOSE`,`ENTRYPOINT`とするのが良さそう。
  * しかし、defaultで`root`を強制するのはやっぱりやめてほしい…
3. 作業中はホストマシンのディスクの空き容量に注意
  * レイヤーが多いと必然的にイメージも増える。
  * Docker開発マシンには256GB,512GBなどのサイズの大きいSSDが望ましい。

## よく使うワンライナー

### ビルドと動作確認

1. `docker build -t username/appname -f role/Dockerfile role/`
  * ディレクトリ構成の項で言及した通り`role`の箇所は`base`,`packages`,`ruby`など、AnsibleやChefのRoleにならったディレクトリパスに。
  * `username/appname`の部分は別に自由でいいが一旦dockerhubのセオリーに則っておく。チーム開発ならusernameのところをorganizationやチーム名などにしてもよい。
  * 末尾の`role/`は当該Dockerfileのあるディレクトリを指定。`.`だとカレントディレクトリ以下の全ファイルを対象にビルド前チェックが走るので、レイヤーごとに個別にディレクトリ指定をしておくのが無難。
2. `docker run --rm -it username/appname /bin/bash`
  * あるいは、開発中はあらかじめDockerfile最下行に`ENTRYPOINT bash`を書いておく。
  * bashが辛ければzshなどに変えてもいいが、buildし直すと意外と時間がかかって面倒くさい。
  * `alias l='ls -lha --color'`などを`ENTRYPOINT`かその手前に入れておくだけでだいぶラクになる。
  * `--rm`を付けておくとコンテナ終了と同時にコンテナを消してくれる、経済的。
3. `docker logs $(docker ps -q)`
  * コンテナIDを取りたいときは`$(docker ps -q)`などサブシェルを使うとわりとラクできる
  * `grep`しつつ複数rmやkillをしたいときは後述のように`sed`や`awk`で。

### お掃除系

1. `docker rm $(docker ps -f status=exited -q)`
  * `Exited`なコンテナを指定して残りを`rm`。
2. `docker rmi -f $(docker images -q -f "dangling=true")`
  * ビルドやタグ付けに失敗したゴミイメージを消す。
3. `docker rmi -f $(docker images | sed 1,1d | egrep -v "app1|app2" | awk '{print $3}')`
  * 残したいイメージを`egrep`で除外して残りを`rmi`。
  * 依存関係にあるイメージは`-f`を指定してもエラーを出して削除対象から除外してくれる、安心設計。

## DockerHub関連

### 準備

* DockerHubのアカウントは作っておく
* privateリポジトリは無料版ではひとつのみ、基本はpublicだと認識しておく
  * DockerHubに限った話ではないが`password`や`credentials`等の取扱いには注意

### セオリーを理解する

* 公開するなら~~`MAINTAINER`~~ `LABEL maintainer "username@example.com"`を必ず入れる
* `username/appname:tag`が基本型
* `:tag`を空欄にすると自動で`:latest`が割り当てられる
* 以上のことから、破壊的変更を加えたときはdockerhubへpushする前にタグを切っておくとよい
  * refs: <http://stackoverflow.com/questions/27643017/do-i-need-to-manually-tag-latest-when-pushing-to-docker-public-repository>

※MAINTAINERは最近deplicatedになったそうなので訂正しました

## References

* [Explore \- Docker Hub](https://hub.docker.com/explore/)
* [Best practices for writing Dockerfiles \- Docker Documentation](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)
* [Dockerfile のベストプラクティス — Docker\-docs\-ja 1\.9\.0b ドキュメント](http://docs.docker.jp/engine/articles/dockerfile_best-practice.html)
* [Dockerのボリュームプラグインとストレージドライバ（Dockerの最新機能を使ってみよう：第2回） \- さくらのナレッジ](http://knowledge.sakura.ad.jp/knowledge/5021/)
* [dockerhub \- do I need to manually tag "latest" when pushing to docker public repository? \- Stack Overflow](http://stackoverflow.com/questions/27643017/do-i-need-to-manually-tag-latest-when-pushing-to-docker-public-repository)
* [1\.13以降はMAINTAINERの代わりにLABELを使うようにしよう \- Qiita](http://qiita.com/okisanjp/items/a43068f956d273703ea8)

