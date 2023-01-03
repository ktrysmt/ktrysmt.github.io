---
layout: post
title: "常駐ClamAV"
date: 2014-02-28 15:09:08 +0900
comments: true
categories: Security
published: true
---

ClamAVを使うときのコピペ用まとめ。

## Install

at CentOS 6

```
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
yum -y install clamd
```

## Configure

```
sed -i "s/User clam/#User clam/" /etc/clamd.conf
/etc/init.d/clamd start
chkconfg clamd on
sed -i 's/^Example/#Example/g' /etc/freshclam.conf
exec $SHELL # 一応
freshclam
touch virusscan.sh
chmod +x virusscan.sh
echo "/proc/" >> ~/clamscan.exclude
echo "/sys/" >> ~/clamscan.exclude
```

## Manual Scan

```
clamscan --infected --remove --recursive <path>
```

## put the shell

### ~/virusscan.sh

```
#!/bin/bash

PATH=/usr/bin:/bin

# clamd update
yum -y update clamd > /dev/null 2>&1
freshclam > /dev/null 2>&1

# excludeopt setup
excludelist=/root/clamscan.exclude
if [ -s $excludelist ]; then
    for i in `cat $excludelist`
    do
        if [ $(echo "$i"|grep \/$) ]; then
            i=`echo $i|sed -e 's/^\([^ ]*\)\/$/\1/p' -e d`
            excludeopt="${excludeopt} --exclude-dir=^$i"
        else
            excludeopt="${excludeopt} --exclude=^$i"
        fi
    done
fi

# virus scan
CLAMSCANTMP=`mktemp`
clamscan --recursive --infected ${excludeopt} / > $CLAMSCANTMP 2>&1
[ ! -z "$(grep FOUND$ $CLAMSCANTMP)" ] && \

# report mail send
grep FOUND$ $CLAMSCANTMP | mail -s "Virus Found in `hostname`" root
rm -f $CLAMSCANTMP

```

## edit crontab

```
0 5 * * * ~/virusscan.sh
```
