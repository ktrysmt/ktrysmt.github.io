---
layout: post
title: "Git Hook・ワークフロー活用術"
date: 2014-02-03 12:00:00 +0900
comments: true
categories: Developer Tools
published: true
description: "Gitの入力補完、post-commitフックによる自動push、pre-commit/post-commitの実用例、git blameによるSRP算出をまとめて解説"
tags:
  - git
  - bash
  - centos
  - devops
  - php
  - shell
redirect_from:
  - /blog/git-input-completion/
  - /blog/setting-git-push-by-hook-post-commit/
  - /blog/for-example-git-hook-of-pre-and-post-commit-command/
  - /blog/calculate-srp-by-git-blame/
---

## BashでGitコマンドやブランチ名を入力補完する

[Git入門 - Gitのコマンド補完](http://www8.atwiki.jp/git_jp/pages/29.html)

上記のサイトに触発されてGit環境改善に乗り出す。

### gitをインストール

以下のサイトを参考に1.8.2をインストール。

[CentOSに最新のgitをインストールする - clavierの日記](http://clavier.hatenablog.com/entry/2013/05/18/204050)

コマンドの記述だけ引用すると以下の通り。

```
# yum install -y zlib-devel perl-devel gettext gcc curl-devel
# wget http://git-core.googlecode.com/files/git-1.8.2.3.tar.gz
# tar xvfz git-1.8.2.3.tar.gz
# cd git-1.8.2.3
# ./configure
# make
# make install
# git --version
git version 1.8.2.3
```

ところが、私の環境がCentOS6.5だったからなのか？、`make install`に失敗した。
ログを見る限り実行ファイルはあるみたいだったので、シンボリックリンクをはって対応。

```
# ln -s /usr/local/bin/git /usr/bin/
```

### 入力補完シェルの場所を確認する

```
$ locate git-completion.bash
/usr/share/doc/git-1.8.2.1/contrib/completion/git-completion.bash
$ locate git-prompt.sh
/usr/share/doc/git-1.8.2.1/contrib/completion/git-prompt.sh
```

今回は上記に保存されていた。

### .bashrcを編集

```
$ vi $HOME/.bashrc
```

.bashrcに以下の記述を追加。

```
source /usr/share/doc/git-1.8.2.1/contrib/completion/git-prompt.sh
source /usr/share/doc/git-1.8.2.1/contrib/completion/git-completion.bash
GIT_PS1_SHOWDIRTYSTATE=true
export PS1='\[\033[1;32m\]\u@\h\[\033[00m\]:\[\033[1;34m\]\w\[\033[1;31m\]$(__git_ps1)\[\033[00m\][\t]$ '
```

記述を反映。

```
$ source $HOME/.bashrc
```

`$(__git_ps1)`という記述、これはカレントのディレクトリがgitリポジトリである場合、カレントのブランチ名を表示する。
Gitのブランチ名を参照できるようになってるのは、以下のサイトを参考にした。

[ターミナルでgitのコマンドを補完したりブランチ名を表示する – macでgitを便利に使うために – - PPl@ce](http://pplace.jp/2013/12/1601/)

ついでに、好みで`[\t]`も追加している。プロンプトにタイムスタンプが欲しいと思うことが多いので。

### 参考
+ [CentOSに最新のgitをインストールする - clavierの日記](http://clavier.hatenablog.com/entry/2013/05/18/204050)
+ [Git入門 - Gitのコマンド補完](http://www8.atwiki.jp/git_jp/pages/29.html)
+ [ターミナルでgitのコマンドを補完したりブランチ名を表示する – macでgitを便利に使うために – - PPl@ce](http://pplace.jp/2013/12/1601/)

## Gitのpost-commitフックでcommit時に自動pushする

ソースコードコミットしたらBitbucketに自動プッシュさせたい。

コミットログとかrebaseとかにあまりこだわらない人向け。

### 確認箇所

+ `push.default`は設定されているか、Gitのバージョンはいくつか
+ 対象となる作業ブランチは目的のリモートブランチにupstream setされているか
+ `git ls-remote`はできるか
+ `.git/hooks/post-commit`は設定されているか

### push.defaultは設定されているか、Gitのバージョンはいくつか

version 1.7.11以降なら`push.default simple`に。

そうでなければ`push.default upstream`にしておく。

``` sh
git config --global push.default simple
```

``` sh
git config --global push.default upstream
```

別に`--global`でなくてもよい。

### 対象となる作業ブランチは目的のリモートブランチにupstream setされているか

ローカルで先行してブランチ作って開発していた場合upstreamが設定されていない。

これに気づかず少し時間を食った。

#### ローカルブランチとリモートブランチを関連付ける

ローカルからみたUpstream先を「追跡ブランチ」と言うらしい。知らなかった…。

追跡ブランチ設定をしたいローカルにチェックアウトし、その後の`git branch -u`で、追跡したい先のブランチ名をリモートリポジトリの名前も含めて引数に指定する。

``` sh
git checkout local-branch
git branch -u origin/remote-branch
```

普通はUpstreamが設定されてないだけで`local-branch`と`remote-branch`は同じ名前のブランチだと思う。

### .git/hooks/post-commitは設定されているか

#### post-commit

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

### commitしてもpushされないときは

だいたい`fatal:`で`no upstream`と出てるので`git branch -u`を行う。

### Bitbucketにパス無しでPushしたいのにできない

むしろコレでかなり時間食った。

結局`ssh-keygen`とデプロイ鍵登録による`ssh://`プロトコルは諦め、`https://`を使ってremote url内に`username:password`を挿入することにした。

パスワード平文で書いてあるのでかなりソワソワする。

``` sh
git remote set-url origin https://username:password@bitbucket.org/teamname/projectname.git
```

誰かBitbucketに鍵認証でDeployするにはどうすればいいか教えてください…。

## よく使う.git/hook

ときどき使うのでコピペ用にまとめ。

### .git/hook/pre-commit

```
#!/bin/bash

# ディレクトリ定義
ROOT_DIR=$(git rev-parse --show-toplevel)
LIST=$(git status | grep -e '\(modified:\|new file:\)'| grep '\.php' | cut -d':' -f2 )

# syntaxエラーがあるファイルはコミット禁止
ERRORS_BUFFER=""
for file in $LIST
do
ERRORS=$(php -l $ROOT_DIR$file 2>&1 | grep "Parse error")
if [ "$ERRORS" != "" ]; then
if [ "$ERRORS_BUFFER" != "" ]; then
ERRORS_BUFFER="$ERRORS_BUFFER\n$ERRORS"
else
ERRORS_BUFFER="$ERRORS"
fi
echo "Syntax errors found in file: $file "
exit 1
fi
done

# masterブランチ、testingブランチへはコミット禁止
branch="$(git symbolic-ref HEAD 2>/dev/null)" || \
"$(git describe --contains --all HEAD)"
if [ "${branch##refs/heads/}" = "master" ]; then
echo "Do not commit on the master branch!"
exit 1
fi
if [ "${branch##refs/heads/}" = "testing" ]; then
echo "Do not commit on the testing branch!"
exit 1
fi
```

### .git/hook/post-commit

```
#!/bin/bash

# master、testingを除くremoteのマージ済みブランチを削除
git push origin $(git branch -r --merged | grep origin/ | egrep -v "(testing|master)$" | grep -v "HEAD" | sed s~origin/~:~);

# コミットと同時にpush(up-streamの必要あり)
git --git-dir=.git push origin
```

### 参考

- <http://tm.root-n.com/unix:command_operation:find_php_lint>
- <http://ktrysmt.github.io/blog/setting-git-push-by-hook-post-commit/>

## git blameでSRP（単一責任原則）を算出する

[git blameによるSRP（単一責任原則）の定量化][L1] が便利だったので早速自分の環境に組み込んで使ってる。

が、計算式をよく見ると、
コミット数・ユーザー数・コードの行数を加算してソートしていてそこだけキモチワルイ。

なのでそれぞれの数字を加算せずそのまま出し、それらを使ってmultiple sortすることにした。

```
function get_SRP() {
  local target_filepath=$1
  local a=$(git --no-pager blame --line-porcelain $target_filepath | sed -n 's/^summary //p' | sort | uniq -c | sort -rn | wc -l)
  local b=( $(cat $target_filepath | wc -l) / 100 - 5)
  local c=$(git --no-pager blame --line-porcelain $target_filepath | sed -n 's/^author //p' | sort | uniq -c | sort -rn | wc -l)
  echo ${a} ${b} ${c} $target_filepath
}

for file in `git ls-files | grep -E '\.js$'`; do
  get_SRP $file
done | sort -k 1,1 -k 2,2 -k 3,3 -nr
```

sort優先順位は個人的な主観で`コミット数 > 行数 > ユーザー数`の順にしている。
また、最近JSをよく触るので計測対象は`.js`にしている。

[L1]:http://ni66ling.hatenadiary.jp/entry/2015/06/25/000444
