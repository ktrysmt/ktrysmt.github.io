---
layout: post
title: "HometypeやめてVromeもやめてVimiumにした"
date: 2015-04-23 15:09:08 +0900 
comments: true
categories: Vim
published: true
---

もはやChromeでブラウジングするのにVimのキーバインドが使えないとかありえないくらいVimに染まってしまった私です。

ChromeをVimのキーバインドで操作したいと思いさがしてみるとhometypeというのに出くわしました。
最初は便利に使ってたのだがその後普段利用する拡張と相性が悪いようで上手く動かなくなり、断念。

ほかにいいのがないか探したところVromeがいいらしく使ってみました。
快適に動作するのだが、今度は普段重用するEvernoteWebクリッパーと相性が悪いことが判明。
VromeがOnだとWebクリッパーがうまく動作しなくなってしまう。

Evernoteヘビーユーザーの私としてはかなり困るので、代替案を考えることになります。

色々デバッグした結果わかったことだが、どうもVromeはCtrlやAltを使ったショートカット（というかキーバインド？）をことごとく持って行ってしまうらしい。
デフォルトで大量のキーバインドをマッピングしてしまうのもちょっと気に入らない。
オプション画面で不要なキーバインドに対してunmapを大量に書かなければいけなくなっていた。



で結局Vimiumに落ち着きました。

- Vimium <https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb>

設定は以下です。

```
# move pixels
pxを60px=>120pxに

# cumstom mapping
map <a-r> closeTabsOnRight
map <a-l> closeTabsOnLeft
map <a-o> closeOtherTabs
unmap <c-y>

# ignore urls
http*://*messenger.com/*
http*://*facebook.com/*
http*://*chatwork.com/*
http*://*slack.com/*
http*://*messenger.com/*
http*://*google.com/*
http*://*feedly.com/*
```

キーボードショートカットが充実しているサイトやGoogle系は除外しました。
個人的にcloseOtherTabsやcloseTabsOnLeftはほぼ必須で、重要。
`<c-y>`はGoogleのShortUrlのキーバインドに使ってるのでunmapしています。


Vimium軽量で快適。

# 参考

- <https://chrome.google.com/webstore/detail/googl-url-shortener/iblijlcdoidgdpfknkckljiocdbnlagk>
- <https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb>
