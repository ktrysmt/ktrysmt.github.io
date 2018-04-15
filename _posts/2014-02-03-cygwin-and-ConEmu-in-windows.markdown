---
layout: post
title: "cygwin＋ConEmuでWinターミナル環境整備"
date: 2014-02-02 15:09:08 +0900 
comments: true
categories: Windows
published: true
---

基本的にWindowsはLinuxな開発にあまり適していないので、さっさとVagrantなりVPSなりにログインしてしまうに限る。
が、それでも最低限のターミナル環境が整っていないと作業効率は落ちてしまう。
しかし、環境整備に時間をかけ過ぎるのも本末転倒。

Cygwinはインストールも簡単ですぐにsshできるので気に入っていたが、minttyにはタブ機能がない。
Console2を使うのがいいようだが、
Cygwinとconsole2は相性が悪いのか、私の環境ではbashうまいことが起動しない。
さて困ったというところでConEmuというターミナルエミュレータを発見。

インストールも簡単だったので設定した項目だけメモ。

#### 追記（2014/09/07）
最近はCmberというターミナルを見つけて、そっちを使っています。ConEmuと殆ど変わらないけれどペーストがShift+InsでなくCtrl+Vでいいので入力作業は多少楽できる。

## Main>Font
VLゴシック、サイズは14に設定。文字は少し小さめで。

## Startup>Tasks

![Startup>Tasks](/images/article/conemu-1.png)

新規にグループを追加（＋ボタン）
MinGW（任意）と入力し、cygwinのbashのパスを入れる。

```
"C:\cygwin64\bin\bash.exe" --login -i
```

Upボタンを連打してMinGWタスクを一番上にもってきておく。
Hotkeyの設定でCtrl+N等にMinGWタスクを割り当てて置くとなお良い。

## Startup

一つ上に戻って、Startup optionsに先ほどのMinGWを設定する。

> Startup options>Specified named task>{MinGW}

ラジオボタンで`Specified named task`を選択し、`{MinGW}`を選ぶ。

## Features>Colors

![Features>Colors](/images/article/conemu-2.png)

カラーコードは自分の見やすいように色々変えられるので変えると良いが、
Schemesでカラーパターンを選べるので、ここから選ぶのが手っ取り早い。
私はubuntuを使用。

## key&Macro>Mark/Copy

選択してすぐクリップボードにコピペできるようにする。

Text selectionをONにし、Alwaysを選択
Copy on Left Button releaseをON

## Features>Transparency
透明化しすぎると見難くなるので、程よい透明度に。

## ウィンドウの分割・クローズ

> CTRL+SHIFT+Eで縦分割
> CTRL+SHIFT+Oで横分割
> WIN+ALT+DELでウィンドウのクローズ

でそれぞれできる。なかなか便利。

## タブを行き来するショートカットの変更

![ショートカットキーの変更](/images/article/conemu-3.png)

Win+番号でタグジャンプするのがデフォルトだが、これだとすこし左手が窮屈で押しづらいので、
ALT+番号に変更する。

ついでに、タブリネームをしやすくするため、タブリネームにALT+Rを割り当てておく。

以下の記事を読んで、たしかにそうだと思って、その影響を受けている。

[事故らないために普段守っているターミナルの運用ポリシ（Mac + iTerm2） | TechRacho](http://techracho.bpsinc.jp/morimorihoge/2014_01_12/15093?utm_source=dlvr.it&utm_medium=twitter)

- ターミナル誤操作を防ぐための習慣として、タブ番号を指定してジャンプし、CTRL+TABでタブ移動しないようにする。
- タブに名前をつけるようにし、かつローカルのホームや母艦を1番目のタブに常に固定しておく。
