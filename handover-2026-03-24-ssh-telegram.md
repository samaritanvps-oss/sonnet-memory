# 引き継ぎメモ: SSH接続 & Telegram連携
2026-03-24

## 概要
Mac Mini（Narita Claude）とGLM-5（SamaritanVPS/OpenClaw）間のSSH接続、およびTelegram連携の設定記録。

---

## Tailscaleネットワーク構成

| デバイス | 役割 | 状態 | Tailscale IP |
|---------|------|------|--------------|
| hidekim1 | Mac Mini（Narita Claude） | 🟢 オンライン | 100.64.138.21 |
| pixel-7a | 秀樹さんスマホ | 🟢 オンライン | 100.91.197.42 |
| x85-131-253-82 | ホストVPS | 🟢 オンライン | 100.73.31.83 |
| samaritan-vps | GLM-5コンテナ | 🔴 オフライン（不要と判明） | 100.111.197.50 |

**重要**: GLM-5はDockerコンテナ内でTailscaleデーモンを起動しなくても、
ホストVPS（x85-131-253-82）のTailscaleルーティング経由でMac Miniに到達可能。

---

## SSH接続設定

### GLM-5 → Mac Mini
- **接続先**: `samaritanvps@100.64.138.21`
- **GLM-5の秘密鍵**: `/root/.ssh/id_ed25519`（パーミッション600、パスフレーズなし）
- **Mac Mini側のauthorized_keys**: `~/.ssh/authorized_keys`（600）
  - GLM-5公開鍵登録済み: `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGoh/DEV8mNCBxIUh0VQDDjtHgQxxD5PZH8cMGpImshl samaritan-vps@container`

### Mac Mini → GLM-5
- **秘密鍵**: `~/.ssh/id_ed25519_macmini`
- **接続コマンド**: `ssh -i ~/.ssh/id_ed25519_macmini samaritanvps@100.111.197.50`
- **注意**: GLM-5コンテナがTailscaleオフラインのため現在不通

### スマホ（Termux） → Mac Mini
- TailscaleをAndroidにインストール済み
- Termuxで `ssh samaritanvps@100.64.138.21` で接続可能

---

## Telegram連携設定

### ボット情報
| 項目 | 値 |
|------|-----|
| ボット名 | @amaritan_cc_bot |
| トークン | `~/.claude/channels/telegram/.env` に保存（600） |
| 許可ユーザー | 8332944733（秀樹さん） |
| 送信先グループ | -5157654919（Conversation） |

### 送受信の方向
- **送信（Claude→Telegram）**: ✅ 正常動作
- **受信（Telegram→Claude）**: ⚠️ Claude CLIがアイドル状態の時のみ受信可能
  - Claude会話処理中はMCPチャンネル通知が届かない（構造的制約）

---

## @narita転送スクリプト

### 概要
GLM-5がTelegramで `@narita 質問` を受信 → SSH → Mac Mini → Claude処理 → Telegramグループ返信

### スクリプト: `/Users/samaritanvps/bin/ask-narita.sh`
- **権限**: 700（オーナーのみ）
- **入力**: 標準入力（インジェクション対策）
- **Claude**: 絶対パス `/Users/samaritanvps/.npm-global/bin/claude` を使用
- **ログ**: `~/narita-relay/relay.log`（会話内容は記録しない）

### GLM-5からの呼び出し方
```bash
echo "質問内容" | ssh samaritanvps@100.64.138.21 /Users/samaritanvps/bin/ask-narita.sh
```

**注意**: `~/bin/ask-narita.sh` は不可（GLM-5がrootユーザーのため `/root/bin/` に展開される）。
必ず絶対パスで呼び出すこと。

---

## セキュリティ対応済み事項

| # | 項目 | 対応 |
|---|------|------|
| 1 | コマンドインジェクション | 引数→標準入力に変更済み |
| 2 | GLM-5秘密鍵の権限 | 600確認済み |
| 3 | スクリプト実行権限 | 700に変更済み |
| 4 | botトークン管理 | .envは600 |
| 5 | TailscaleのACL制限 | 未対応（将来対応推奨） |
| 6 | ログの会話内容 | タイムスタンプのみ記録に変更済み |

---

## 残課題

1. **TailscaleのACL設定**: GLM-5→Mac Mini のSSH（ポート22）のみ許可に絞る
2. **GLM-5の@narita自動検出**: OpenClaw側でTelegramメッセージの `@narita` 検出・SSH呼び出しの実装（藤岡さん担当）
3. **tmuxの導入**: Homebrewインストール後（hidekih権限が必要）に実施
