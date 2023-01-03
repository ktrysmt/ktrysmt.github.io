---
layout: post
title: "サーバにスワップがないのでスワップファイルを作成する"
date: 2014-10-18 15:09:08 +0900
comments: true
categories: Linux
published: true
---

たまに必要になるのでコピペ用にメモ。

## 作成

```
sudo dd if=/dev/zero of=/swapfile bs=1024K count=1024 # 1GB欲しい場合
sudo chmod 0600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

## スワップの確認

```
 swapon -s
```

## /etc/fstabに追記

```
sudo bash -c 'echo "/swapfile               swap                    swap    defaults        0 0" >> /etc/fstab'
```
