---
layout: post
title: "CoreOS+Goの組み合わせはたしかに面白そうだ"
date: 2015-04-05 15:09:08 +0900 
comments: true
categories: Node
published: true
---

以下のエントリがとても興味をくすぐった。

- <http://tdoc.info/blog/2015/02/17/go_with_docker.html>

kochaを使ってDockerHubなどナシにコンテナ運用しちゃおうという試み。

CoreOSさえ使いこなせれば、あとはデプロイのフローをどうするか考えるだけになる。
S3やDropboxに上げればいいのかな。

インストールが楽だという理由とまずはDockerになれるという理由でboot2dockerを使っていたが、
VagrantでCoreOSのVMを使ってCoreOS慣れしておくほうがよさそう。

DockerMeetup#4でもあったように、

- CoreOS本体
- fleet
- etcd
- Terraform
- cloud-config

このへんはひと通り慣れておく必要がある。

というかGolangをまだあまり触ってないのでこれもチュートリアルやっとこう。
バイナリ一本でいけるkochaが気楽で良さそうだ。そいやrevelが早いと一時期噂になったな。

- <http://naoina.github.io/kocha/docs/>

APIサーバというニーズにGoはすごくマッチしてると思う。

着手優先度的にはまずGoだが、
boot2docker触ってて思ったがコンテナが一瞬で立ち上がるのは想像以上に爽快というか楽しいので、
CoreOSは結構楽しみながらいろいろ覚えられていいなと感じてる。

んでフロントはreact.jsで気持ちよく。


SEO？RoRとか、PHPならSlim+Guzzleとかでいいんじゃないかな。

- <http://www.slimframework.com/>
