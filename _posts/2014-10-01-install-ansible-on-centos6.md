---
layout: post
title: "CentOS6でAnsibleを使う"
date: 2014-10-02 15:09:08 +0900
comments: true
categories: Ansible
published: true
---

環境はCentOS 6.5 x86_64です。

## install repository of epel

```
$ wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ sudo rpm --upgrade --verbose --hash epel-release-6-8.noarch.rpm
```

## install ansible

```
$ sudo yum install ansible --enablerepo=epel
$ ansible --version
ansible 1.7
```

## markupsafe モジュールが足りない

yumで入れたのだがmarkupsafeが無いと言われたので入れる。

```
$ sudo pip install markupsafe
```

## create hosts.ini and playbook.yml


```
$ vim hosts.ini

[www]
x.x.x.x
[image]
y.y.y.y
[www:vars]
db_host=z.z.z.z
db_name=database_name
db_user=user
db_pass=password
server_host=abc.abc.abc.abc
```

`[www]`や`[image]`はグループを指す。

変数を使いたい場合は`[グループ名:vars]`以下に定義すればplaybook.yml内で使える。


``` yaml
$ vim playbook.yml

---
- hosts: www
- sudo: true
- tasks:
    - name: be sure something is installed
      shell: yum -y install something
    - name: use vars
      shell: >-
        sed -i "s/#ServerName www.example.com:80/ServerName \{\{ server_host }}:80/" /etc/httpd/conf/httpd.conf;
    - name: restart
      shell: /etc/init.d/httpd restart
```

hosts.iniでで定義した変数は`\{\{ 変数名 }}`で呼び出せる。

## ファイルを分割してincludeしたい

ansible公式にベストプラクティスあがってるけど。

個人的な用途としてはそんなに仰々しくなくても、単にincludeするだけで十分。

ディレクトリ構成はこんなふうに。

```
├ hosts.ini
├ playbook.yml
└ includes
    ├─ mysql.yml
    └─ php55.yml
```

playbook.yml内でincludeモジュールを使ってファイルを呼ぶだけでいい。

include元のplaybook

``` yaml
$ vim playbook.yml

---
- hosts: web
  sudo: true
  tasks:
    - include: includes/php55.yml
```

includeされるyamlファイルは以下。

``` yaml
$ vim includes/php55.yml

---
- name: setup epel
  shell: rpm -Uvh http://dl.fedoraproject.org/pub/epel/5/x86_64/epel-release-5-4.noarch.rpm;
  ignore_errors: True
- name: setup remi
  shell: rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-5.rpm;
  ignore_errors: True
- name: edit epel repo
  shell: sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo
- name: setup php 5.5
  shell: yum -y install --enablerepo=remi-php55 php php-mysql php-mb* php-devel php-pear php-pecl-apc php-dom php-json;
```

## ssh-keygenした？

あらかじめ、実行元サーバから各種実行先サーバ群へ鍵認証でSSHできるようにしておくこと。

## 実行してみる

```
ansible-playbook playbook.yml -i hosts.ini
```

## Dry-Run

`--check`をつける。

```
ansible-playbook playbook.yml -i hosts.ini --check
```

## 任意のコマンドを展開することも可能

状態チェックしたいときに便利。

`www`や`image`に当たる引数にはそのiniファイルのグループを指定。

``` sh
ansible image -a "cat /etc/redhat-release" -i hosts.ini
ansible www -m shell -a "ps aux | grep httpd" -i hosts.ini
```

`-m`でモジュール指定できる。パイプでワンライナー書くならshellモジュールで。
