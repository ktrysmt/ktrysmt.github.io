---
layout: post
title: "どのようにテスト駆動開発を現場に導入していくか"
date: 2014-09-20 15:09:08 +0900
comments: true
categories: Test
published: true
---

結論を言うとプロジェクトの状況に合わせ、テストに対する考え方や使い方・アプローチの仕方を臨機応変に対応するのが望ましいようだ。

大きく分けると2パターンある。

+ 既に稼働しているプロジェクトのソースコードで、テストコードがない
+ 新しいプロジェクトでソースコードはこれから書く

## 既に稼働しているプロジェクトのソースコードで、テストコードがない

BDD

+ Rspec, capybara, selenium

## 新しいプロジェクトでソースコードはこれから書く

TDD

+ xUnit, xspec

## 参考

+ <http://modeverv.hateblo.jp/entry/20130324/1364143010>
+ <http://tkengo.github.io/blog/2014/01/28/capybara-testing-on-php/>

PHPなプロジェクトでphpunit入れにくい構成でもRspecとcapybaraを使えるとなれば、つぶしが利くんじゃないか。
Ruby(Rails)も今のトレンドのひとつだし、投資効果が高いなと思った。
