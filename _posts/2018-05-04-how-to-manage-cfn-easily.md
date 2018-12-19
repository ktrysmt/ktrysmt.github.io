---
layout: post
title: "疲弊しないためのCloudFormation管理手法"
date: 2018-05-05
comments: true
categories: AWS
published: true
use_toc: true
description: "私も日々お世話になっている大変便利なCFnですが、上手に付き合うにはいくつかコツがいるのかなぁと感じたので、要点と管理手法をまとめました。"
---


つらいところ
------------

### NestedStackがつらい
まずStackを事前にS3へ上げないといけないというのが...。
S3Bucketももれなくワンオフで管理したい身としては、stack間の前後関係はなるべく排除したいのです。
また、個人的な信条として、stack間の依存を許容するのはIAMリソース程度に留めたいです。
そんなわけで`aws cloudformation packange`も、CFnの管理という点においては嬉しくなく...。
S3を介するのでなくstackテンプレート間の相対パスの解決を、手元でやってほしかったです。

### Outputの取り扱いが難しい
NestedStackを諦めるとすると、テンプレートを分割する場合、相互参照はおおむねOutput経由で参照することになってきます。
しかしOutputは、本来CFnをコントロールする権限しか持たないIAMが特権的にクレデンシャル等自身の権限範囲外の情報を読み取ることができてしまったりします。取り扱いは慎重に行う必要があります。

Outputに関連してCFnにはImportValueという仕組みがあり、これを使えば他のStackが特定のOutputに依存していることを知らずに当該Stackを編集・削除しようとた場合でも、ロックを掛けて更新の拒絶をしてくれます。コレ自体は便利でよいのですが、顧客が欲しかったものは依存関係の崩れに事前に気付けるような参照性や閲覧製の高い仕組みで、依存関係の崩れを検知してブロックしてくれる仕組みではなかったと思うのですよね。ImportValue以前に、Stack間のハードコーディング自体を設計思想として許容しない感じにしてほしかった...。

Outputの一覧性も低いこともあり、全体的にデリケートな印象で、個人的にはあまり使いたくないです。

### update-change-setは無い
というかそもそもなぜchange-setというAPIなのか...。APIデザインはterraformのようにしてほしかったですね。create -> plan -> applyでよかった。
change-setにはhistoryの概念がなく、結果update-change-setなるものも無いです。
それに近いことを実現するならStackに複数のchange-setをぶら下げ自前でナンバリングしてくようなアプローチになりますが、
どのみちexecしてみないとどんなFAILが出るかわからない以上、実行してFAILを受け取るほうが早く、chnage-setを複数管理する動機も生まれません。

### FAILがこわいがマネジメントコンソールへ行くのも面倒
ただでさえ何でFAILするのかわからない、FAILしても原因がすぐにはわかりにくかったりするのがCFnです。
テンプレートをバージョン管理していたとしても、コード上のdiffよりは実際のStack上のdiffとしての差分確認をしておきたいところです。

対策を考える
------------

### Nestはしない、世界は1つ（1枚）
1stack=1yamlでいきます。yamlが巨大化していく問題はテンプレートエンジンの力を借ります。
DRYにしたいという要望に対しては、CFnのParametersとテンプレートエンジンを組み合わせて対処します。
CFnのテンプレートを包括的に管理するPreProcesserタイプのOSSもいくつかありますが、
yamlを単純に分割してるだけという単純さがトラブル時の原因切り分けで効いてくるので、
PreProcesserではなくテンプレートエンジンを選択しています。

### 極力Outputはしない、必要ならARNを経由する
なるべく1stack内でリソースIDの相互参照が完結するようにします。

ひとつだけ例外だと思っているのがIAMリソースです。
アクセス権限の発行は特別に管理することで権限まわりのミスを防ぎたい狙いがあります。
ミスると悪用や情報漏えいのリスクを抱えることになるので。

Stackをまたいで参照させたい場合は、極力Arnを経由します。

VPCに対するEC2の管理においては単一VPC配下のEC2の数と種類が増えがちなことからOutputとImportValueの組み合わせが効きそうに思えますが、個人的にこれは悪手だと思っています。アプリケーションの用途や目的ごとにVPC自体をわけたほうがネットワークセグメント的に見てトラフィックの輻輳やウィルスの拡散等させにくくなりますし、トラフィックの流れがVPC内で単純になりログの参照性も上がり、また、アウトバウンドの制御や遮断もしやすくなります。

### stackに紐付くchange-setは常に1つ
change-setを1stack内に複数ぶら下げたところで管理が煩雑になるだけでメリットが無いので、1stack=1change-setを原則にします。
change-setの変更に際してはdelete->createと毎回作り直す感じで行きます。

### 実際のリソース(stackとchange-set)の差分を取るようにする
テンプレートをバージョン管理するのは当然としても、バージョン管理上のdiffだけでは心もとないので、`get-template`を使って実リソースとしてCFnに登録されているテンプレートのdiffを見るようにします。

マネジメントコンソールを見るのにくらべて取れる情報量は見劣りしますが、awscliで実リソースを参照している確実性と、ターミナル上(CIの画面上)でdiffとして出力できる気楽さがよいです。
まぁいざFAILしたときはマネジメントコンソールを見に行くんですが。diffチェックだけではFAILを防ぎきれないので、この点は仕方なしです。

対策していく
--------------

疲弊しないために、なんとかコードで対策していきます。

### Requirements

現状、以下のツールを使ってworkflowを組み立てています。

* jinja (python)
* cfn-lint (nodejs)
* make

インデントを保持しながらファイルマージしたいがために、テンプレートエンジンにはjinjaを採用しています。
cfn-lintはcloudformation validate-templateだけのlintだけでは不安だったので導入しました。
これらについてMakeを基点にオペレーションしていきます。

### Directory structure

ディレクトリ構成はこんな感じです。`.git`や`.node_modules`等は省略しています。

```
.
├── .cache
│  ├── .gitignore
│  ├── a.diff
│  └── b.diff
├── .gitignore
├── bin
│  └── bundle.py
├── dist
│  ├── .gitignore
│  └── privileged-access
│     └── bundle.yaml
├── Makefile
├── package.json
├── readme.md
├── requirements.txt
├── src
│  └── privileged-access
│     ├── param.create-change-set.json
│     ├── param.create-stack.json
│     ├── root.yaml.j2
│     ├── tmpl.addition.yaml.j2
│     ├── tmpl.group.yaml.j2
│     ├── tmpl.inline-policy.yaml.j2
│     ├── tmpl.managed-policy.yaml.j2
│     ├── tmpl.role.yaml.j2
│     └── tmpl.user.yaml.j2
└── yarn.lock
```

src/以下にディレクトリを作成します。このディレクトリ名がほぼそのままStackNameになります。
ディレクトリの他、以下のファイルがオペレーションにあたり最低限必要となるファイルです。

* `src/ディレクトリ名/root.yaml.j2`
* `src/ディレクトリ名/param.create-stack.json`
* `src/ディレクトリ名/param.create-change-set.json`

上記の例では`privileged-access`というディレクトリを作りました。
パラメータはMakefile内において`--cli-input-json`経由で渡すようにしますので、これも作成しておきます。

### Workflow

`make`でだいたいのことをやります。

`make create/stack Target=ディレクトリ名`というふうに引数でディレクトリ名を指定します。
なおディレクトリをネストさせても各種オペレーションが通るようにMakefile内でちょっと工夫しています（後述）。

`make bundle` ではAWSリソース環境ごとに異なる名前を使いたい場合のために、`Org`というパラメータが使えるようになってます。
`make bundle Target=sample Org=production` というように呼びます。
これを使うとJinjaテンプレート内で `｛｛ org ｝｝` というパラメータ変数を埋め込んでおけば、そこに`produciton`の文字列がbundle時に挿入されます。
特にS3Bucketなどは全体で一意なため、パラメータ変数を使って重複を避けてつつリソースをまたいで同じテンプレートが使えるように工夫します。

このように、管理規模が大きくなってきた場合を出来る限り想定して構成しています。

おおまかな作業の流れは以下です。

1. ディレクトリ作成後`root.yaml.j2`をまず作成します、このファイルを起点としてテンプレートを記述、分割していきます。
2. テンプレートが仕上がったら`make bundle`を叩きます。すると`bin/bundle.py`が呼ばれ、jinjaでテンプレートを結合します。結合後のファイルは`dist/ディレクトリ名/bundle.yaml`に出力されます。
3. `make lint`でbundle後のファイル`dist/ディレクトリ名/bundle.yaml`に対し`aws cloudformation validate-template`と`cfn-lint`をかけます。
4. `make create/stack`や`make create/change-set`でStackテンプレートをアップロードします。
5. `wait/stack-create`などで正常終了を待ちます。change-setの場合は`diff/change-set`でStackテンプレートのdiffをダイナミックに出力します。
6. change-setの場合は`exec/change-set`で実行します。その後また例によって`wait`したりします。



### Sources

以下Makefileの一部です。ソース全体は[こちらのリポジトリ(ktrysmt/j2cfn-boiler)][1]にアップロードしてあります。

[1]: https://github.com/ktrysmt/j2cfn-boiler

```makefile
.DEFAULT_GOAL := help
THIS_FILE := $(lastword $(MAKEFILE_LIST))

# env
Target=
StackName=$(shell echo "${Target}" | perl -pe 's%/%-%g')
ChangeSetName=change-set-$(StackName)
# filename
BundleFile=bundle.yaml
RootFile=root.yaml.j2
ParamCreateStackFile=param.create-stack.json
ParamCreateChangeSetFile=param.create-change-set.json

setup: ## install dependencies w/ npm and python. ## -
	@if [ "" != `which yarn` ]; then yarn; \
		elif [ "" != `which npm` ]; then npm i; \
		else echo "Please install nodejs."; fi
	@if [ "" != `which pip3` ]; then pip3 install -r ./requirements.txt; \
		elif [ "" != `which pip` ]; then pip install -r ./requirements.txt; \
		else echo "Please install python3."; fi

bundle: init ## bundle partial templates into one ## make bundle Target=privileged-access
	@if [ ! -f ./src/${Target}/root.yaml.j2 ]; then echo "it is not found that './src/${Target}/root.yaml.j2'." && exit 1; fi
	@if [ "" != `which python3` ]; then OutputFile=$(BundleFile) RootFile=$(RootFile) python3 ./bin/bundle.py $(MAKEFLAGS); \
		elif [ "" != `which python` ]; then OutputFile=$(BundleFile) RootFile=$(RootFile) python ./bin/bundle.py $(MAKEFLAGS); \
		else echo "Please install python3."; fi

lint: init ## lint the template ## make lint Target=privileged-access
	aws cloudformation validate-template --template-body file://dist/${Target}/$(BundleFile)
	cfn-lint validate ./dist/${Target}/$(BundleFile)

create/stack: init ## call create-stack ## make create/stack Target=privileged-access
	aws cloudformation create-stack \
		--stack-name ${StackName} \
		--cli-input-json file://src/${Target}/$(ParamCreateStackFile) \
		--template-body file://dist/${Target}/$(BundleFile)

create/change-set: init ## call create-change-set ## make create/change-set Target=privileged-access
	aws cloudformation create-change-set \
		--stack-name ${StackName} \
		--change-set-name ${ChangeSetName} \
		--cli-input-json file://src/${Target}/$(ParamCreateChangeSetFile) \
		--template-body file://dist/${Target}/$(BundleFile)

wait/stack-create: init ## call wait stack-create-complete ## make wait/stack-create Target=privileged-access
	aws cloudformation wait stack-create-complete \
		--stack-name ${StackName}

wait/stack-update: init ## call wait stack-update-complete ## make wait/stack-update Target=privileged-access
	aws cloudformation wait stack-update-complete \
		--stack-name ${StackName}

wait/change-set-create: init ## call wait change-set-create-complete ## make wait/change-set-create Target=privileged-access
	aws cloudformation wait change-set-create-complete \
		--change-set-name ${ChangeSetName} \
		--stack-name ${StackName}

desc/change-set: init ## call describe-change-set ## make desc/change-set Target=privileged-access
	aws cloudformation describe-change-set \
		--change-set-name ${ChangeSetName} \
		--stack-name ${StackName}

exec/change-set: init ## call execute-change-set ## make exec/change-set Target=privileged-access
	aws cloudformation execute-change-set \
		--change-set-name ${ChangeSetName} \
		--stack-name ${StackName}

diff/change-set: init ## check stacks differences ## make diff/change-set Target=privileged-access
	@aws cloudformation get-template --stack-name ${StackName} --output text --query "TemplateBody" > .cache/a.diff
	@aws cloudformation get-template --stack-name ${StackName} --change-set-name ${ChangeSetName} --output text --query "TemplateBody" > .cache/b.diff
	@diff -U 3 .cache/{a,b}.diff || :

init:
	@if [ "" = "${Target}" ]; then echo 'The "Target" environment is empty. ex) `make lint Target=privileged-access` ' && exit 1; fi
```

`root.yaml.j2`の中身は以下のような感じです。

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: ''

# Parameters:
  # nothing.

Resources:

  # User
  {％ filter indent(2) ％}{％ include "./tmpl.user.yaml.j2" ％}{％ endfilter ％}

  # Group
  {％ filter indent(2) ％}{％ include "./tmpl.group.yaml.j2" ％}{％ endfilter ％}

  # Addition of user to group
  {％ filter indent(2) ％}{％ include "./tmpl.addition.yaml.j2" ％}{％ endfilter ％}

  # ManagedPolicy
  {％ filter indent(2) ％}{％ include "./tmpl.managed-policy.yaml.j2" ％}{％ endfilter ％}

  # Role
  {％ filter indent(2) ％}{％ include "./tmpl.role.yaml.j2" ％}{％ endfilter ％}

  # Inline Policy
  {％ filter indent(2) ％}{％ include "./tmpl.inline-policy.yaml.j2" ％}{％ endfilter ％}

# Outputs:
  # nothing.
```

（markdownの関係で`%`が全角になってます）

jinjaのテンプレートタグがyamlに埋め込まれてます。
特徴はjinjaの`filter indent(number)`です。yamlはpython同様インデントの数と位置が非常に重要ですが、このfilterオプションを使えば親テンプレート側のインデントを保持しながら子テンプレート内の記述を展開でき、キレイにマージできます。


以下はinclude先の子テンプレートファイルの一例(`./tmpl.role.yaml.j2`)です。こちらは普通にリソース定義を書いているだけです。もちろん子のほうで更にテンプレートタグを使うことも出来ます。

```yaml
UserRoleA:
  Type: 'AWS::IAM::Role'
  Properties:
    RoleName: 'user-role-A'
    Path: '/'
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: 'Allow'
          Principal:
            AWS: 'arn:aws:iam::999999999999:root'
          Action:
            - 'sts:AssumeRole'
          Condition:
            Bool:
              'aws:MultiFactorAuthPresent': 'true'
```

あまり凝った記述をするのは管理コストが上がってしまうのでおすすめしませんが、テンプレートエンジンの力を借りて即時変数をyaml内で宣言したり、制御構文を埋め込んだりもできます。

変数を埋め込む場合はテンプレート内で `｛｛ var ｝｝` のように宣言して、`make bundle Target=xxx var=1234` というように同名の引数を渡してあげればOKです。

所感
--------

Makefileが少々煩雑なのは残念ですが、極力素朴な実装にしたかったのでこうなりました。

あくまで1stack=1yamlなので構造が単純で、余計な依存関係を気にする必要がない点はよさそうかなと思っています。
機能単位で全てのリソースを依存のない1ファイルにまとめられると、クロスアカウントでのテストがしやすくなるのが助かります。

jinjaを採用した理由は、前述の通りインデントを保持してキレイにマージできた点です。
cfn-lintを使う関係上最初はnodejsのテンプレートエンジンを使おうとしたのですが、
jinjaの`filter indent`に相当する機能を持つものがどうも見当たらず、jinjaに落ち着きました。

そもそもInfrastructureAsCodeをterraformにするかCFnにするのかでも悩んでいたのですが、tfもtfで色々とバグを踏んで辛かったのと、上記の通りうまく管理できさえすればAWSリソースも広くカバーされているので、AWSを使う限りはCFnを使うほうがメリットは多いかなと考えています。

ただCFnにも微妙なハマりポイントがいくつかあり、なかなか悩ましいです。これについては別途まとめていこうかと思っています。
