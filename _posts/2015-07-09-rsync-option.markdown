---
layout: post
title: "rsyncで押さえておくオプション"
date: 2015-06-28 15:09:08 +0900
comments: true
categories: Linux
published: true
---

バックアップ、転送、ネットワークまたいで簡単に差分チェックと、便利なrsync。
最近あんま使わなくなってきたけどやっぱりまだたまに使うのでメモ。

### 権限まわりを上書きしない

```
rsync -avz --no-o --no-p --no-g
```

転送先になんらか制約があるときに。

### Dry-Run

```
rsync -avzn src/ dst/
```

`n`つけるだけでいいので楽。かなり便利

### 転送先のファイルの更新日時が転送元より未来だったら上書きしない

```
rsync -avzu src/ dst/
```

Work In Progress な状況でもファイルを送りつけたいときに。

### 鍵変更

```
rsync -avz -e "ssh i /path/to/key" src/ hostname:/dst/
```

普通のsshとちょっと書き方が違うので注意。

### その他

`exclude`や`include`は普通すぎるので割愛。

`avz`は基本つけときゃ間違いないが、cronで使ったり標準出力捨てるならvは取っていい。
