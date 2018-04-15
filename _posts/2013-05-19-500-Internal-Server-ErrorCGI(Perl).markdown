---
layout: post
title: "CGI(Perl)が 500 Internal Server Error のときの対処法いろいろ"
date: 2013-05-19 12:48:21 +0000
comments: true
categories: Perl
published: true
---

[<img class="aligncenter size-full wp-image-208" alt="wg8v" src="http://labo.in-ception.com/wp-content/uploads/2013/08/wg8v.png" width="769" height="161" />][1]  
500エラーは何が原因なんだか検討もつかず途方に暮れます。

[1]: http://labo.in-ception.com/wp-content/uploads/2013/08/wg8v.png

ググればいろいろなサイトで対処法が載っていますが、  
後輩が困ってたので自分用も兼ねつつのまとめです。

他の手段も覚えたら随時更新します。

## Syntax Check

基本の基本ですが、ファイルに構文エラーがないか確認しておきます。

<pre lang="sh"># perl -c filename.cgi</pre>

## おまじないハイフン

ソースコードの冒頭を確認します。

<pre lang="sh">#!/usr/bin/perl -
...code...</pre>

サーバによってはperlのパスが違うこともあるので以下も試します。

<pre lang="sh">#!/usr/bin/local/perl -
...code...</pre>

ハイフンふたつとかも試してみる。

<pre lang="sh">#!/usr/bin/perl --
...code...</pre>

## ファイルの文字コードを確認

Perlでは、Sjift_JISでは特定のマルチバイトがきたとき、うまく動きません。  
古いPerl環境の場合は、ソースに漢字・日本語が混じるときはいったん`EUC-JP`にしてみると幸せになれそうです。  
テキストエディタで保存する際、文字コードをEUCにして保存するとよいです。

## 改行コードの変更

Windows端末でCGIファイルを作成していたり、上書き保存していたりしたら要確認。  
CRLFである場合はCR消しましょう。LF残しで。  
消すツールはなんでもいいですが、nkfコマンドがラク。

<pre lang="sh"># nkf -Lu filename.cgi</pre>

## エラーメッセージを見よう

そこまでやったら、サーバのエラーログを確認します。
