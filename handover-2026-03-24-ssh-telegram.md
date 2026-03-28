# 引き継ぎメモ: SSH接続 & Telegram連携
2026-03-24（最終更新: 2026-03-29）

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

### Tailscaleトラブル時の復旧手順（2026-03-29 実績）

Tailscaleが停止して接続不能になることがある。以下で復旧する：

```bash
/Applications/Tailscale.app/Contents/MacOS/Tailscale up --reset
/Applications/Tailscale.app/Contents/MacOS/Tailscale status
```

`hidekim1` の状態が `offline` → `-`（アイドル）に変わればOK。

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

### sshdトラブル時の復旧（2026-03-29 実績）

sshdが停止してSSH接続不能になることがある。
**System Settings → General → Sharing → Remote Login → オン** で有効化できる。

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
GLM-5がTelegramで `@narita 質問` を受信 → SSH → Mac Mini → Anthropic API呼び出し → Telegramグループ返信

### スクリプト: `/Users/samaritanvps/bin/ask-narita.sh`
- **権限**: 700（オーナーのみ）
- **入力**: 標準入力（インジェクション対策）
- **モデル**: Claude Sonnet 4.6（`/Users/samaritanvps/bin/ask-narita-api.mjs` 経由でAPI直接呼び出し）
- **ログ**: `~/narita-relay/relay.log`（会話内容は記録しない）

### 認証について（重要）
- `claude -p` コマンドはSSHセッションからmacOSキーチェーンにアクセスできないためOAuth不可
- `ANTHROPIC_API_KEY`（`narita-poster/.env`）を使ってAnthropicAPIを直接呼び出す方式に変更済み
- APIクレジットが不足すると "Credit balance is too low" でエラーになる → console.anthropic.com で補充

### GLM-5からの呼び出し方
```bash
echo "質問内容" | ssh samaritanvps@100.64.138.21 /Users/samaritanvps/bin/ask-narita.sh
```

**注意**: `~/bin/ask-narita.sh` は不可（GLM-5がrootユーザーのため `/root/bin/` に展開される）。
必ず絶対パスで呼び出すこと。

---

## スリープ対策（2026-03-27 更新）

### 問題
`caffeinate -i`（アイドルスリープのみ防止）では深夜にMaintenance Sleep（DarkWake）が発生し、
SSH接続が深夜0時〜朝5時頃に切断される問題があった。

### 解決策
`caffeinate -s`（システムスリープ自体を防止）に変更。`~/.zprofile` に設定済み：

```bash
pgrep caffeinate || caffeinate -s &
```

手動で再起動する場合：
```bash
pkill caffeinate && caffeinate -s &
```

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

## インフルエンサー活動チェック機能（2026-03-25追加）

### 概要
成田関連インフルエンサー12アカウントの過去24時間のX投稿を自動収集し、Claude Haikuでレポート生成してTelegramグループに送信する。

### 対象アカウント（12件）
| username | ラベル |
|----------|--------|
| unarikun_narita | うなりくん（成田市公式マスコット） |
| shimurayusuke | シムラユウスケ（ふわりの森） |
| 1991mazda787B | 1991mazda787B（成田空港航空写真） |
| KEITAROinNRT | けーたろー（成田在住・地域活動） |
| HOUEICOFFEE | HOUEI COFFEE（成田山門前カフェ） |
| furusho_charlie | 古庄チャーリィ（プロ航空カメラマン） |
| ikarosairline | 月刊エアライン編集部 |
| nari_map | なりまっぷ（成田市情報発信） |
| miyatch_camera | ミヤッチカメラ |
| naritarisa | 成田梨紗 |
| naritacity_ | 成田市公式（観光・文化・スポーツ） |
| narita_fudou | 大本山成田山新勝寺公式 |

### ファイル構成
| 項目 | パス |
|------|------|
| スクリプト | `/Users/samaritanvps/narita-poster/scripts/narita_influencer_check.mjs` |
| launchd plist | `/Users/samaritanvps/Library/LaunchAgents/com.samaritanvps.narita-influencer-check.plist` |
| ログ | `/Users/samaritanvps/narita-poster/logs/influencer_check.log` |

### 処理フロー
1. Twitter API v2（Bearer Token / App-only）でユーザーIDを一括解決
2. 各アカウントの過去24時間のツイートを取得（リツイート除く、最大10件）
3. Claude Haiku（`claude-haiku-4-5-20251001`）でレポート生成
4. Telegram グループ（-5157654919）に送信

### 実行スケジュール
- **毎日 18:00 JST**（launchd）
- 手動実行: `node /Users/samaritanvps/narita-poster/scripts/narita_influencer_check.mjs`

### 既知の問題・注意点
- **Anthropic APIクレジット不足時**: Haiku が HTTP 400 を返すが、スクリプトはエラーログなしに `（レポート生成失敗）` にフォールバックする
- Twitter API Bearer Tokenは読み取り専用（App-only）のため投稿には使えない

---

## 残課題

1. **TailscaleのACL設定**: GLM-5→Mac Mini のSSH（ポート22）のみ許可に絞る
2. **GLM-5の@narita自動検出**: OpenClaw側でTelegramメッセージの `@narita` 検出・SSH呼び出しの実装（藤岡さん担当）
3. **tmuxの導入**: Homebrewインストール後（hidekih権限が必要）に実施
4. **influencer_check.mjs のエラーハンドリング強化**: Haiku API 400エラーをログに明示的に記録する