---
layout: post
title: "「git blameでSRP算出」が便利"
date: 2016-04-23 15:09:08 +0900 
comments: true
categories: Git
published: true
---

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
