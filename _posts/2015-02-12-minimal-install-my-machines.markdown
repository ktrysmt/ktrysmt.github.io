---
layout: post
title: "開発マシンにはVagrant,Browser,Terminalがあればいい"
date: 2015-02-12 15:09:08 +0900
comments: true
categories: Vagrant
published: true
---


というのは言いすぎだが。

手持ちのWindowsをOSクリーンインストールするようなことになったときのためにメモ。

## 必需品

厳密には、

- Vagrant（VirtualBox）
- Browser（Chrome一択）
- keyhac（組み合わせキーが使えるキーリマップツール）
- PuTTY（軽くて安定なターミナル）
- Clibor（使いやすい、クリップボード監視）
- AntiVirusソフト（Avira、MS純正等、好みで）
- Font（Migu M+、VLゴシック、Inconsolata等）
- かざぐるマウス（マウスジェスチャ）

削って削って、これくらいか。

あとはホストマシンがネットにさえ繋がっていれば開発には困らない。

## 仕事でいるかも

次点で、場合によっては仕事で必要になりそうなものが以下。

- FTPクライアント（Filezillaとか）
- Fiddler（ローカルプロキシ）
- Wireshark（パケットキャプチャ）
- Git bash（ローカルからSSHが必要な場合に）
- Skype（連絡）
- LightShot（スクショ、Gyazoでも）
- クラウドストレージ（Dropbox,OneDrive,MEGA,Box等）

## たまに使う

以下は個人的な趣味。なくてもいい。

- VirtualWin（仮想デスクトップ、壁紙切替Moduleが良い感じ）
- CCleaner（潔癖症）
- MPC（たまに動画見る）
- Cmder（Git Bashあるならなくてもいいが、綺麗。ConEmu2より使いやすい）

## 余談

いまんとこMacは持ってないが、もし使うようになったとしても同じようなソフトウェアがググればあると思われる。それらを入れれば作業には困らないはず。
Ubuntu使うとしたらVirtualBoxでなくlxcを使えるようになるのでちょっと気になる、が設定するのがめんどくさいので手は出していない。
