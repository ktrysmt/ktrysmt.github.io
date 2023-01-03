---
layout: post
title: "[Git] git/hooks/post-commitで、commitしたら自動でリモートにPushする"
date: 2014-10-01 15:09:08 +0900
comments: true
categories: Git
published: true
---

ソースコードコミットしたらBitbucketに自動プッシュさせたい。

コミットログとかrebaseとかにあまりこだわらない人向け。

### 確認箇所

+ `push.default`は設定されているか、Gitのバージョンはいくつか
+ 対象となる作業ブランチは目的のリモートブランチにupstream setされているか
+ `git ls-remote`はできるか
+ `.git/hooks/post-commit`は設定されているか

## push.defaultは設定されているか、Gitのバージョンはいくつか

version 1.7.11以降なら`push.default simple`に。

そうでなければ`push.default upstream`にしておく。

### コマンド

``` sh
git config --global push.default simple
```

``` sh
git config --global push.default upstream
```

別に`--global`でなくてもよい。

## 対象となる作業ブランチは目的のリモートブランチにupstream setされているか

ローカルで先行してブランチ作って開発していた場合upstreamが設定されていない。

これに気づかず少し時間を食った。

### ローカルブランチとリモートブランチを関連付ける

ローカルからみたUpstream先を「追跡ブランチ」と言うらしい。知らなかった…。

追跡ブランチ設定をしたいローカルにチェックアウトし、その後の`git branch -u`で、追跡したい先のブランチ名をリモートリポジトリの名前も含めて引数に指定する。

``` sh
git checkout local-branch
git branch -u origin/remote-branch
```

普通はUpstreamが設定されてないだけで`local-branch`と`remote-branch`は同じ名前のブランチだと思う。

## .git/hooks/post-commitは設定されているか

### post-commit

最初はファイルがないと思うのでcpして作る。

``` sh
cd .git/hooks/
cp post-commit.sample post-commit
vim post-commit
```

処理したい内容を書けばいい。リモートにPushしたいだけなら、

``` sh
git --git-dir=.git push origin
```

でいい。set upstreamしておけば大丈夫だけど、意図しないリモートサーバに送るというのはすごく困るので、いちおう`origin`指定まではしておく。

## commitしてもpushされないときは

だいたい`fatal:`で`no upstream`と出てるので`git branch -u`を行う。

## Bitbucketにパス無しでPushしたいのにできない

むしろコレでかなり時間食った。

結局`ssh-keygen`とデプロイ鍵登録による`ssh://`プロトコルは諦め、`https://`を使ってremote url内に`username:password`を挿入することにした。

パスワード平文で書いてあるのでかなりソワソワする。

``` sh
git remote set-url origin https://username:password@bitbucket.org/teamname/projectname.git
```

誰かBitbucketに鍵認証でDeployするにはどうすればいいか教えてください…。
