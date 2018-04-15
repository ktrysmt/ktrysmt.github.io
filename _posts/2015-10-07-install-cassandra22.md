---
layout: post
title: "Cassandra v2.2をCentOS6で使う"
date: 2015-10-07 15:09:08 +0900 
comments: true
categories: Cassandra
published: true
---

## Install

Cassandra 2.1以降はJava7,8をサポートしているので特にバージョン番号気にせずyumを使います。

```
# yum -y install java
# echo '[datastax]
name = DataStax Repo for Apache Cassandra
baseurl = http://rpm.datastax.com/community
enabled = 1
gpgcheck = 0' > /etc/yum.repos.d/datastax.repo 
# yum -y install dsc22 cassandra-tools
# chown -R cassandra:cassandra /var/lib/cassandra
# exec $SHELL # reload
# service cassandra start
```

## Control nodes

nodetoolでノードの管理ができる

```
# nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Tokens       Owns    Host ID                               Rack
UN  127.0.0.1  211.42 KB  256          ?       09c9d240-a5ee-4cea-95c5-b0a7fd015320  rack1

Note: Non-system keyspaces don't have the same replication settings, effective ownership information is meaningless
```

## Operation CQL

データ操作はcql。cqlshがインストールされている

```
# cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 2.2.2 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cqlsh> 

```
