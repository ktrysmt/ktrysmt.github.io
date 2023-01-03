---
layout: post
title: "ThinkPad X1 CarbonのCPUが遅い"
date: 2014-01-23 05:52:07 +0000
comments: true
categories: Windows
published: true
---

購入してからそれほど日が経っていないのに、Chromeのタブをたくさん開くとすぐに遅くなる。
Laptopだしもともとそういうもの？と思い込んでしまいChromeExtensionをあれこれ使ったりと試行錯誤。
でも抜本的解決にはならず困っていた。

結論を言ってしまうと、これは「電源オプション」の「省電力モード」が原因だった。

持ち歩くことが多いので電源オプションにて省電力モードを選んでいたことが良くなかった。
モードで決めてしまうと、CPUPowerの負荷増大時のCore SpeedのHz数に、
ボーダーラインが敷かれてしまうようだった。

バッテリー駆動時ならまだいいのだが、
省電力モードを選んでいたことにより電源コードにつないでいるときでもCPUのパワーセーブが働き、
ボーダーラインを超えないようになっていた。

windowsのLaptopならどの端末も以下の方法でCPUのパワーセーブを変更できる。

![コンパネ](/images/article/cpupower-win7_1.png)

コントロールパネルから電源オプションを選択。

![モード選択](/images/article/cpupower-win7_2.png)

任意のモードを選択、使ってるものならなんでもいい。

![高度な設定](/images/article/cpupower-win7_3.png)

詳細な電源設定の変更を選択。

![CPUPower](/images/article/cpupower-win7_4.png)

Intel(R) Graphics Settings を Maximum Performance に変更。

高度な設定では他にもバッテリーが残り10%になったら何かするとか細く設定ができる。
「オーバークロックしなきゃいけないのか？」とか思い始めていたから、気づいてよかった。
以下記事が参考になった。

- [http://d.hatena.ne.jp/nekop/20130122/1358822977](http://d.hatena.ne.jp/nekop/20130122/1358822977)
- [http://superuser.com/questions/663568/how-to-limit-my-cpu-power-programmatically-on-windows-7](http://superuser.com/questions/663568/how-to-limit-my-cpu-power-programmatically-on-windows-7)

Windowsで正確にマシンのスペックを把握するには、CPUZを使うといいようだ。
- [CPUZのダウンロード](http://www.filehippo.com/jp/download_cpuz)
