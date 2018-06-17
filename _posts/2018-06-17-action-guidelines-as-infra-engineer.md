---
layout: post
title: "私がインフラエンジニアとして働くにあたって大切にしている基本的な考え方"
date: 2018-06-17 15:00:00 +0900
comments: true
categories: "Management"
published: true
use_toc: true
description: "インフラエンジニアリングの現場も重要な意思決定の連続ですが、日々の設計やツール・サービス選定にあたって意識している考え方があったので、言語化しました。" 
---


価値観
----

### 価値観の軸

* 安全性
* 堅牢性
* 効率性

どんな現場でも通じるようにするためには多すぎないほうがいいと思ったので、3つに絞りました。

### それぞれの観点
* 安全性
  * 技術スタックの鮮度管理
  * 外部攻撃や内部犯行等に対する適切な検知・ブロック・セグメンテーションの仕組みづくり
  * 事故発生時の適切な保全・応急処置策の策定
* 堅牢性
  * 過不足のないリソース配分・コスト最適化
  * 障害発生時の迅速な復旧・修復
* 効率性
  * 一貫性・再現性・冪等性をもった基盤管理体制
  * 各種作業の自動化推進・基盤提供

堅牢性については、特にオートスケーリングなどの技術を指して弾力性と表現したりもします。
効率性については、ほとんどCIによる自動化やコンテナ化を指します。プロダクトや現場によっては複雑な業務プロセスの効率化という切り口もありそうです。


### 価値観に優先順位を定める

<img src="/assets/images/blog/action-guidelines-as-infra-engineer/image.png" style="float:right;width:50%;">

1. 安全性
2. 堅牢性
3. 効率性

優先順位がないとただのスローガンになってしまい判断の参考にならず意味を成さないので、必ず順序を決めます。

だいたいいつもこんな感じの図を描いてます。

### 優先順位の考え方

* 安全性と堅牢性であれば安全性を選ぶ
* 堅牢性と効率性であれば堅牢性を選ぶ

ときには安全性と堅牢性をセットで考えなければならないケースもあるので一概には言えませんが、おおよそはこのような価値観に基づいて行動します。
「多少ナイーブな実装であっても安全性を犠牲にするのはありえないので今はこの実装でいきましょう」というような考え方をします。

これは、アーキテクチャ設計やツール選定だけでなく日々のタスク消化の優先順位にも当てはめます。
着手のしやすさや個人的な趣味趣向を優先して効率性に関するタスクばかりやるのではなく、安全性や堅牢性のタスクを優先的に処理しましょう、といった具合です。
言葉にすればごく当たり前なことなのですが、実行に移すには案外強い意志が必要なものです...。チームメイトとの認識合わせ・協力関係も重要です。

世界観
------------

### 何年かかっても実現したいと思うような、目指すべき世界

* 安全で頑丈な基盤・システムをユーザー（エンドユーザーおよびアプリケーションエンジニア）に提供しつづける
* 基盤／システムを支えるにあたり、物量（人員や手作業）ではなく、技術を拠りどころとする
* インフラキャパシティに関する完全な無人化・オートメーション化 ... etc

SRE本におけるトイルが完全になくなることはおそらく無いのでしょうが、それでもある程度理想郷が描ける・共有できている方がチームにとっていいと思うので、世界観についても言及するようにしています。

Vision-Mission-Values
-----------------------

前述の価値観と世界観に、更に所属組織のビジネスモデルからくる当面のミッションを追加して、Vision-Mission-Valuesとセットで定義するといいと思います。
中長期的な技術目標を立てるにあたっての、いい指針になってくれます。
私自身、今の現場ではこの考え方に基づいてインフラエンジニアリングとしてのVision-Mission-Valuesを定義し、日々のスケジューリングに役立てています。

補足：オリエンタルランドのSCSE
----

ベースとなる枠組みとしてオリエンタルランド社の行動規準「SCSE」を参考にし、当てはめました。

SCSEの内容は以下です。

> 【  Safety  】	 安全な場所、やすらぎを感じる空間を作りだすために、ゲストにとっても、キャストにとっても安全を最優先すること。  
> 【Courtesy】	 “すべてのゲストがVIP”との理念に基づき、言葉づかいや対応が丁寧なことはもちろん、相手の立場にたった、親しみやすく、心をこめたおもてなしをすること。  
> 【  Show  】	あらゆるものがテーマショーという観点から考えられ、施設の点検や清掃などを行うほか、キャストも「毎日が初演」の気持ちを忘れず、ショーを演じること。  
> 【Efficiency】	安全、礼儀正しさ、ショーを心がけ、さらにチームワークを発揮することで、効率を高めること。

社会人になりたての頃に社のビジネス書を読んで、この考え方に大変な感銘を受けたのをよく覚えています。
なによりも優先順位がついていることが素晴らしい。
経営陣の考え方を現場レベルにまで落とし込み、かつ現場が自律的に行動できるようになり、マネジメントコスト削減にもつながっていくのだと思います。

恣意的な判断で設計が決まってしまう現場
---------------------------------------

受託開発・自社サービスなどいろいろな現場で仕事をしてきましたが、どこにおいても、エンジニア個人の好みや恣意的な判断で各種ツールや言語・アーキテクチャを選定する現場がいかに多かったか。そのことが今でも気になっており、また自身もそうならないように自戒を込めてこの記事を書きました。

参考
-----

* <http://www.olc.co.jp/ja/csr/5daiji/management/safety/scse.html>
* <https://www.slideshare.net/bbbart/mission-vision-and-strategy-in-organisations>