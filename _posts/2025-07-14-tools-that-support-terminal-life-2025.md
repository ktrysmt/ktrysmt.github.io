---
layout: post
title: "2025版 ターミナル引きこもり生活を支えてくれる道具たち"
date: 2025-07-14 09:00:00 +0900
categories: Zsh
published: true
use_toc: false
---

過去記事で部分的に解説してきた情報のとりまとめ・アップデートを兼ねて、2025年7月時点での生活を支える道具類を紹介します。

(参考：過去記事)
* <https://ktrysmt.github.io/blog/summary-of-my-recent-llm-activities/>
* <https://ktrysmt.github.io/blog/changed-init-vim-to-init-lua/>
* <https://ktrysmt.github.io/blog/migrate-zinit-to-sheldon/>
* <https://ktrysmt.github.io/blog/finish-work-early-with-cli-made-by-rust/>
* <https://ktrysmt.github.io/blog/snippets-management-by-peco-and-zsh/>


## CLI
- [eza](https://github.com/eza-community/eza) ... ls代替。exaのfork
- [ripgrep](https://github.com/BurntSushi/ripgrep) ... grep代替。だいぶ市民権を得た
- [fd](https://github.com/sharkdp/fd) ... find代替。AI Agent のおかげでこれもかなり聞かれるようになった
- [bat](https://github.com/sharkdp/bat) ... cat代替。見やすい
- [delta](https://github.com/dandavison/delta) ... リッチdiff。`git diff | delta`としてもいいし`.gitconfig`いじって組み込んでも良い
- [tig](https://github.com/jonas/tig) ... 昔からあるgit log viewer。最近は [lazygit](https://github.com/jesseduffield/lazygit) が覇権だが、stage/unstageはvim上でやるので未だにこれ
- [fzf](https://github.com/junegunn/fzf)/[fzy](https://github.com/jhawthorn/fzy)/[peco](https://github.com/peco/peco) ... これがないとかなりつらい。とにかくラク。`CTRL_T` に当てたりhistoryなどの絞り込みに
- [jnv](https://github.com/ynqa/jnv) ... 絞り込み検索しつつjqのqueryをプレビューできる。たまに使うが複雑なjsonに出くわしたときは正直もうLLMに聞くほうが速くて効率が良くなってしまった。もうあまり使わないかも。
- [trash](https://formulae.brew.sh/formula/trash) ... ゴミ箱系コマンド。復元できるならこれでなくてもなんでもいい。事故防止に。
- [difit](https://github.com/yoshiko-pg/difit) ... PR作るほどでもない、個人でLLMとの対話のみでやってるときのセルフレビューで。
- [gh](https://cli.github.com/manual/) ... pull requestの番号で手元に落とせるのでレビューのとき便利。それ以外の用途だとLLMに雑にgithubいかせたいときに。
- [tmux](https://github.com/tmux/tmux) ... もはや乗り換えるのがだるいだけっていう。
- [neovim](https://neovim.io/) ... プラグイン/言語別のツールまで言い出すと膨大な量になってしまうのでそのへんは今回は割愛。
- [claude](https://docs.anthropic.com/en/docs/claude-code/overview) ... 救世主。

## Package Manager
- [brew](https://formulae.brew.sh/) ... homebrew/linuxbrew。結局いまだにこれ。
- [sheldon](https://github.com/rossmacarthur/sheldon) ... zsh管理は概ねこれで済ませている。あいかわらず、速いは正義。
- [mise](https://github.com/jdx/mise) ... asdfから乗り換え。nodeやrubyの管理はこれでやってる。速い。
- [uv](https://github.com/astral-sh/uv) ... pythonだけはmiseではなくuvで管理、どうしてもみんながvenvに依存するので。poetory→Ryeの流れに加えLLM界隈で急速に市民権を得たことがやはり大きい。
- [pnpm](https://pnpm.io/ja/) ... mise経由でいれるが、npmより動作が安定していて速いので使えるならこっちを使いたい。プロジェクトによっては使えないのがつらいが、仕方ない。

## GUI

### 共通：クロスプラットフォーム
- [wezterm](https://wezterm.org/) ... クロスプラットフォームで設定がlua。フォント周りの微調整がしやすい。キーバインドも多少いじれる。
- [vscode](https://code.visualstudio.com/) ... markdown editorもしくはLLMインターフェイスとして。あんま使ってない。
* [obsidian](https://obsidian.md/) ... markdown editorの本命。技術系テキストの溜め込み先としても使う、git管理。
* [notion](https://notion.so) ... 非技術系の個人のナレッジの溜め込み先として。クロスプラットフォームでモバイルが優秀なのがでかい。
* [clibor](https://chigusa-web.com/clibor/) ... win/mac用両方あるクリップボード管理。大変使いやすいのと、さっとスニペットに切り替えできるのとFIFOモードがよい。
* [vagrant](https://developer.hashicorp.com/vagrant)/[virtualbox](https://www.virtualbox.org/) ... 低レイヤー触るときでクラウドにサーバ立てるのがだるいとき、結局一番使いやすいのがこれ。vbguestのgemのバグ、もうだれも直してくれないのだろうか。。
### macOS
* [raycast](https://www.raycast.com/) ... グローバルホットキーでブラウザやterminalをfrontmostにしたり。あまり凝った使い方はしてないがかなりカスタマイズしやすい。
* [alttab](https://alt-tab-macos.netlify.app/) ... raycastも使うがすぐウィンドウを切り替えたいときはこっち。手癖になっているのと、chromeを２つあげてるとしてchrome同士を切り替えたいため。
* [rectangle](https://rectangleapp.com/) ... Win+left,Win+rightの手癖がそのまま再現できる。
* [karabiner-elements](https://karabiner-elements.pqrs.org/) ... 説明不要のキーマップカスタマイズ。macosメジャーバージョンアップを急ぐと壊れる可能性があるのでメジャーを上げるのは慎重に。macosに関してはhammerspoonのほうがluaで設定を書けるので自由度は上なのだが、たまに固まるのでやはり安定のkarabinerに落ち着く。
* [linermouse](https://linearmouse.app/ja-JP/) ... マウスのポインタの動きの細かい制御など。加重が苦手なので線形にしたり移動距離を大きくしたり。
* [macgesture](https://github.com/MacGesture/MacGesture) ... マウスジェスチャー。定番。
### Windows
* [flow launcher](https://www.flowlauncher.com/) ... win用ランチャ。powertoysrunと同じwoxのforkらしい。powertoysrunより少し動作が軽いのとUIをいじれるのでこちらを採用。ueliもいいがflow launcherのほうが軽快だった。
* [x-mouse button contorl](https://www.highrez.co.uk/downloads/XMouseButtonControl.htm) ... win用。マウスにマクロをセットできる。ホイール長押しでmacでいうミッションコントロール（ウィンドウを俯瞰で表示）にしたり、追加ボタンにキーマップやマクロを割り当てたり。
* [keyhac](https://sites.google.com/site/craftware/keyhac-ja) ... win/mac両方あるようだが、使うのはwinのみ。winだと安定して動くキーマップカスタマイズ。設定がpythonなので設定がしやすい。autohotkeyのほうが自由度はやや上な気がするがpythonで書けるのがラクで助かるので長らくこっち。
* [f.lux](https://justgetflux.com/) ... 時間だけでなくアプリによってON/OFFができるのでOS標準ではなくこっちを使い続けている。


## おわりに

vimrc盆栽もそうですが、道具は適宜手入れをしたり入れ替えたり気にかけるようにしてみると生活に少し彩りを添えてくれるのでおすすめです。
