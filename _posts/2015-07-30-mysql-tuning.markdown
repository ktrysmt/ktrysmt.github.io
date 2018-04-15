---
layout: post
title: "MySQLの基本的なチューニングまとめ"
date: 2015-07-09 15:09:08 +0900 
comments: true
categories: MySQL
published: true
---

## 基本計算式

```
 innodb_buffer_pool_size 
 + key_buffer_size
 + max_connections * (sort_buffer_size + read_buffer_size + 2MB)
```

## max_connections

物理メモリをもとに基本計算式で計算。

AWS（RDS）では、`DBInstanceClassMemory`の3/4が自動的に割り当てられる。

## wait_timeout

```
mysql> set global wait_timeout=180;
mysql> show global variables like 'wait_timeout';
+---------------------------------------------------+----------+
| Variable_name                                     | Value    |
+---------------------------------------------------+----------+
| wait_timeout                                      | 180      |
+---------------------------------------------------+----------+
```

/etc/my.cnfに書くなら

```
[mysqld]
wait_timeout = 180
```

## `innodb_buffer_pool_size`

DBサーバ専用なら、載ってるメモリの3/4程度は割り当てて良い。

## `key_buffer_size`, `sort_buffer_size`

通常低すぎかも。数百ＭＢ当ててよい。


# まとめ

## InnoDBの場合
----------------

`innodb_buffer_pool_size`には当てられるだけメモリを当てる。
専用サーバなら全体の80%を。共存してるなら他のサービスと相談。
`innodb_log_file_size`は、`innodb_buffer_pool_size`の1/2にする。デフォが2枚組みだから。

`innodb_log_file_size`を変更したら、mysqlを一旦落として、
`/var/lib/mysql/`以下の`idlogfile`を`rm`しておく。で、mysql再起動。

`key_buffer_size`
「一般的には、マシンのメモリ使用率 25 % の値であることが好ましい」一方で
「Key_reads/Key_read_requests の比率は、0.01 より小さいことが望ましい」つまり
99.9%がキャッシュされていることが望ましいということ。

## 調査コマンド(コピペ用)
-------------------

```
-- [Important]
-- innodb_buffer_pool_size + key_buffer_size +
-- max_connections * (sort_buffer_size + read_buffer_size) + max_connections * 2 MB
show variables like "%innodb_buffer%";
show variables like "%innodb_log%";
show variables like "%key_buffer_size%";
show variables like "%max_connections%";
show variables like "%sort_buffer_size%";
show variables like "%read_buffer_size%";

-- ・方法１
-- Key_reads / Key_read_requests =「キャッシュミスレート」
-- これが、1%を下回るように。
-- ・方法２
-- 100 - ( ( Key_reads / Key_read_requests ) * 100 )＝Key Efficiency(%)
-- これが、90% を下回る事が無いように。
show status like "%Key_reads%";
show status like "%Key_read_requests%";

-- Key_blocks_not_flushed：変更されたが、ディスクへのフラッシュはまだされていないキーキャッシュのキーブロック 
-- Key_blocks_unused ：キーキャッシュの未使用ブロックの数 
-- Key_blocks_used ：キーキャッシュのブロックの使用数。この値は、これまで同時使用したブロックの最大値 
-- [適切なkey_buffer_sizeを知るために]
-- Key_block_unused
-- これが数百ブロックといった小さな値になっている場合には増やす必要があります。
-- Key_read_requests / Key_reads 比
-- 100を大きく割り込んでいる値が記録されている場合、key_buffer_sizeは不足。
-- 体感では、Key_blocks_usedは限りなくゼロに近い方がいい。blockは恐らくディスクに近いことと思う。
show status like "%Key_blocks%";
```
