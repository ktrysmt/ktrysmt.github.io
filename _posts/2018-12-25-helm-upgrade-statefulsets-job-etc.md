---
layout: post
title: 'helmでのstatefulsetsの更新をうまくやるワークアラウンド'
date: 2018-12-25
comments: true
categories: Kubernetes
published: true
use_toc: true
description: 'helmの設定を書き換えようとしてもそれがstsやpoddisruptionbudgets.policy、job等についての変更の場合、うまく更新できず困っていたのですが...。'
---

そういう場合は、実podは残しつつstsやpoddisruptionbudgets.policy等の設定のみを消すことで helm upgrade が叩けるようです。
kubectlでのマニュアルオペレーションでなくhelmベースでの管理を継続できるのが主なメリットです。

## helmの状況

[Issue](https://github.com/helm/helm/issues/2149)はあがっていて2.9で一度解決した？ようにみえるのですが、手元の環境も含め2.11時点ではうまくFixされていないようです。

なおこのやり方はGCPの[ジョブの更新についてのドキュメント](https://cloud.google.com/kubernetes-engine/docs/how-to/updating-apps?hl=JA#updating_a_job)にも言及されている方法なので、一定の安心感があります。

## コマンド例

やり方ですが、一旦、

```
$ kubectl delete sts $NAME --cascade=false
$ kubectl delete poddisruptionbudgets.policy $NAME --cascade=false
```

などと `--cascade=false` を付けつつdeleteしてあげると、その後 `helm upgrade --recreate-pods` で更新できるようになります。

## リファレンス

* <https://github.com/helm/helm/issues/2149>
* <https://cloud.google.com/kubernetes-engine/docs/how-to/updating-apps?hl=JA#updating_a_job>
