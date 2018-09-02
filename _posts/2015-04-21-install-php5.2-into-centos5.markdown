---
layout: post
title: "CentOS5にPHP5.2を入れなければならない"
date: 2015-04-19 15:09:08 +0900
comments: true
categories: PHP
published: true
---

遭遇するたびにググってて時間の無駄なのでコピペ用にまとめ。

```
echo '[utter] \
name=Jason’s Utter Ramblings Repo \
baseurl=http://www.jasonlitka.com/media/EL$releasever/$basearch/ \
enabled=0 \
gpgcheck=1 \
gpgkey=http://www.jasonlitka.com/media/RPM-GPG-KEY-jlitka' > /etc/yum.repos.d/utterramblings.repo
yum --enablerepo=utter -y install php php-devel php-mbstring php-mysql php-xml php-gd php-mcrypt mysql mysql-server
```

`yum remove php* mysql*`してから行う場合はmysql-libs消すときの巻き込みに注意。
postfixとcrontabsもremoveされてしまう。
