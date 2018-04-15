---
layout: post
title: "Elixir で Please ensure your locale is set to UTF-8 …"
date: 2016-04-26 15:09:08 +0900 
comments: true
categories: Elixir
published: true
---

最近Elixirを触り始めているのでその内容。  
ホストに直接brewで入れるでなくVagrantで立てた仮想マシン内部に入れる場合によくエラーが出るので直す。

#### エラー文

```
warning: the VM is running with native name encoding of latin1 which may cause Elixir to malfunction as it expects utf8. Please ensure your locale is set to UTF-8 (which can be verified by running "locale" in your shell)
```

#### How to fix

```
echo 'LANG="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_CTYPE="en_US.UTF-8"' >> ~/.zshrc
```

