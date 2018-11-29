---
layout: post
title: 'powered_cdをもうちょっと便利にする'
date: 2018-11-29 00:00:00 +0900
comments: true
categories: 'Zsh'
published: true
use_toc: false
description: 'powered_cd w/ fzf が大変便利で日々お世話になっているのですが、古いディレクトリパスをログから削除する術がなかったので、ちょっとコードをいじりました。'
---


ディレクトリが既に削除されている場合は`cd`せずにログから削除します。

```sh
: "powered_cd" && {
  function chpwd() {
    powered_cd_add_log
  }
  function powered_cd_add_log() {
    local i=0
    cat ~/.powered_cd.log | while read line; do
    (( i++ ))
    if [ i = 30 ]; then
      sed -i -e "30,30d" ~/.powered_cd.log
    elif [ "$line" = "$PWD" ]; then
      sed -i -e "${i},${i}d" ~/.powered_cd.log
    fi
  done
  echo "$PWD" >> ~/.powered_cd.log
  }
  function powered_cd() {
    if (which tac > /dev/null); then
      tac="tac"
    else
      tac="tail -r"
    fi
    if [ $# = 0 ]; then
      local dir=$(eval $tac ~/.powered_cd.log | fzf)
      if [ -d "$dir" ]; then
        cd "$dir"
      else
        local res=$(grep -v -E "^${dir}" ~/.powered_cd.log)
        echo $res > ~/.powered_cd.log
        echo "powerd_cd: deleted old path: ${dir}"
      fi
    elif [ $# = 1 ]; then
      cd $1
    else
      echo "powered_cd: too many arguments"
    fi
  }
  _powered_cd() {
    _files -/
  }
  compdef _powered_cd powered_cd
  [ -e ~/.powered_cd.log ] || touch ~/.powered_cd.log
  alias c="powered_cd"
}
```

元ネタはこちら；

- [cdの履歴を保存してpecoとかfzfとか使ってディレクトリ移動する - Qiita](https://qiita.com/arks22/items/8515a7f4eab37cfbfb17)

