---
layout: post
title: "Obsidian Mobileの同期をGit PluginからRemotely Save + WebDAVに乗り換えたら快適になった"
date: 2026-03-24 09:00:00 +0900
categories: [Developer Tools]
published: true
description: "Obsidian MobileのGit Plugin（isomorphic-git）が遅い問題を、Tailscale + rclone WebDAV + Remotely Saveで解消。systemd常時起動の設定、初回同期時のconflict対処法も。"
tags:
  - obsidian
  - tailscale
  - webdav
  - rclone
---

## Git Pluginが遅い理由

Obsidian MobileのGit Pluginは、内部的にJavaScriptでgitをエミュレート（isomorphic-git）してるらしい。ネイティブのgitバイナリを呼んでいるわけではないので特に大量のファイル/巨大ファイル/履歴が多いリポジトリとの相性が悪い。

Tailscaleが張り巡らされている環境であれば、WebDAV使って母艦から直接配信するのが最も簡便で効率がよかった、Remotely SaveプラグインならObsidian側の設定もまぁまぁシンプル。

## 構成

### 母艦側: rclone

```bash
rclone serve webdav . \
  --addr $(tailscale ip -4):8080 \
  --user your_username \
  --pass your_password \
  --include 'dir/**' \
  --filter '- **'
```

* `--addr $(tailscale ip -4):8080` : TailscaleのIPにのみバインド。ipハードコードでもOK。
* `--include 'dir/**'` と `--filter '- **'` : includeで共有したいディレクトリだけを指定し、filterで残り全てを除外。帯域節約のため。
* `--user` / `--pass` : Tailscale網内に閉じているとはいえ、最低限のBASIC認証はかけておきましょう。

systemdなどで常時起動する場合Tailscaleの接続完了を待ってからバインドする。`tailscale ip -4` は接続前だとエラーを返すので、これをポーリングすれば待ち合わせになる。

```bash
#!/bin/bash
until tailscale ip -4 >/dev/null 2>&1; do sleep 1; done
TS_IP=$(tailscale ip -4)
exec rclone serve webdav /path/to/vault \
  --addr "${TS_IP}:8080" \
  --user your_username \
  --pass your_password \
  --include 'dir/**' \
  --filter '- **'
```

`After=tailscaled.service` でtailscaledの起動後に実行されるが、デーモン起動=接続完了ではないのでポーリング

`exec` でシェルプロセスをrcloneに置き換えるため、systemd が rclone のプロセスを直接管理するようになる。

あとはこいつを `/usr/local/bin/rclone-webdav.sh` などに置いて `chmod +x`

systemd unit は以下

```ini
# /etc/systemd/system/rclone-webdav.service
[Unit]
Description=rclone WebDAV server for Obsidian
After=network-online.target tailscaled.service
Wants=network-online.target tailscaled.service

[Service]
Type=simple
ExecStart=/usr/local/bin/rclone-webdav.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```


```bash
sudo systemctl daemon-reload
sudo systemctl enable --now rclone-webdav.service
```

### Obsidian側: Remotely Save

| 項目 | 設定値 |
|------|--------|
| Remote Service | WebDAV |
| Server Address | `http://<母艦のTailscale IP>:8080/` (末尾スラッシュ必須) |
| Username | rcloneで設定したユーザ名 |
| Password | rcloneで設定したパスワード |
| Abort Sync If Modification Above Percentage | 100 |
| Sync Direction | お好みで（後述） |

#### Abort Sync If Modification Above Percentage

同期時に変更・削除されるファイルの割合が閾値を超えた場合に同期を中断する安全装置。デフォルトは50%。例えばVaultに200ファイルあって同期で150ファイルが変更される場合（75%）、50%を超えるため同期が中断される。

大量削除などの事故を防ぐための機能だが、初回同期や2台目のデバイスとの初回接続では全ファイルが「変更」扱いになるため、ほぼ確実に引っかかる。自分で管理しているWebDAV環境であれば100（実質無効）にしておいて問題ない。

私は複数のモバイルで母艦から配信されるファイルを閲覧したい（readonly）構成なので、100%としてる。

#### Sync Direction

| 設定値 | 用途 |
|--------|------|
| Bidirectional (default) | 双方向同期。複数端末で編集する場合 |
| Incremental Pull And Delete | 母艦をマスターとし、モバイル側は読み取り専用的に使う場合 |

mobileからも編集したい場合はbidirectionalにするとよい。
ほかにも pullするけどDeleteまではしない、とかもある。

前述の通り多端末で母艦からpullする形をとるので `Incremental Pull And Delete`。

#### 初回同期時のconflict

2台目のデバイスなど、既にリモートにファイルがある状態で初めて同期すると `conflict_created_then_do_nothing` で全ファイルがスキップされることがある。これはRemotely Saveが同期メタデータを持っていないため、ローカルとリモートのどちらが正かを判断できず安全側に倒した結果。

Remotely Saveの設定画面下部にある「Delete Sync Metadata」でメタデータをリセットしてから再同期すれば解消する。

## おわり

とても快適です。
