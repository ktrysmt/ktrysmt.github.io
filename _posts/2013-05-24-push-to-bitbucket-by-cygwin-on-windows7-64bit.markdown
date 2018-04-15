---
layout: post
title: "bitbucketにcygwin（windows7 64bit）からpushする"
date: 2013-05-24 12:48:21 +0000
comments: true
categories: Git
published: true
---

bitbucketへcygwinからpushするときのまとめです。リポジトリへアクセスするプロトコルやらツールの使い方やらでググった結果の、自分の環境用ということで一応まとめ。

以下の前提で進めます。

1. bitbucket.orgのアカウントを持っている。  
2. cygwinがインストールされている。

<!--more-->

## cygwinの実行ファイル mintty.exe に管理者権限を与える

これができていないとcygwinからpingやwgetなど、名前解決ができません。  
実行ファイルを右クリックし、プロパティの互換性 から、管理者権限を付与しましょう。

実行ファイルは、cygwinをデフォルトインストールしている場合は大抵下記にあります。

<pre lang="sh">C:cygwinbinmintty.exe</pre>

完了したらcygwinを起動している場合には一度cygwin窓を閉じて、再度開きます。管理者権限で開くときの認証が出るはず。  
ping google.co.jpして、応答が返ってくることを確認します。

## cygwinで鍵生成

さっそく鍵を作ります。

<pre lang="sh">cd /home/.ssh/
ssh-keygen # すべて空EnterでOK
ls -la</pre>

すると、以下のように秘密鍵と公開鍵が確認できるはずです。

<pre lang="sh">id_rsa
id_rsa.pub</pre>

id_rsaをcatで出力、コピペしてbitbucketの鍵追加画面で追加しましょう。

## gitリポジトリをremoteにpush

あとはリポジトリをpushするだけです。remote addする際のアドレスにだけ注意しましょう。  
形式はSSHを選択し、ssh://などは付けないです。付けるといいよ風のblog記事がいくつかありましたが、なくてOKでした。  
以下は入力例です。

<pre lang="sh">git remote add origin git@bitbucket.org/test-apps.git
git push -u origin master</pre>
