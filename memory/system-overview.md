# system-overview.md — Samaritan システム全体像

最終更新: 2026-03-22
出典: Google Drive「samaritan_handover_2026-03-17」「Samaritan_Security_Review_for_Cowork」

---

## 1. エージェント構成（現状）

| 呼称 | 動作場所 | 記憶 | 役割 | 状態 |
|------|---------|------|------|------|
| Root（個人契約Claude） | セッションベース / Mac Miniアクセスなし | セッション限り | 設計思想・監修 | 稼働中 |
| Claude Cowork（私） | Mac Mini / SamaritanVPS専用契約 | セッション限り（CLAUDE.md参照） | 実務支援・実行監督 | 稼働中 |
| GLA-5（Samaritan-GLA5） | VPS / OpenClawインスタンス | /workspace/memory/に永続 | 投稿・巡回・実行 | **停止中** |

**接続ハブ**: samaritanvps@gmail.com（Google OAuth）が全サービスの中心。
**GLA-5のTelegramボット**: @Samaritan0402_bot（秀樹さんへの報告に使用、現在停止中）

---

## 2. GLA-5について

### 概要
GLA-5は[OpenClaw](https://github.com/openclaw/openclaw)（オープンソース自律AIエージェント）をVPS上で常時稼働させたインスタンス。
OpenClawはTelegram等をUIとしてLLMと連携し、ウェブ閲覧・ファイル操作・シェルコマンド実行が可能。
（元名称: Clawdbot → Moltbot → OpenClaw。Moltbookと同じ作者 Peter Steinberger 製）

### 停止経緯
- 2026年3月初頭にセキュリティ侵害を受け停止。復旧見込み未定
- 根本原因: samaritanvps@gmail.com への一極集中（Drive・Gmail・GitHub・Notion全てがこのOAuthに依存）
- このアカウント侵害 → 全システムへ連鎖露出

### GLA-5が担っていたタスク
- **成田多言語投稿** → **Mac Mini M1（launchd）に移行済み**
- Moltbook活動（3時間ごとの関与）→ **Claude Coworkのスケジュールタスクに移行済み**
- Gmail監視（件名`[COWORK]`の下書きキュー）→ 現在未稼働
- Telegram報告（@Samaritan0402_bot）→ 現在未稼働

---

## 3. Narita Poster（Mac Mini移行後の現状）

### 稼働場所
`/Users/samaritanvps/narita-poster/` / launchd（3時間ごと）

### 投稿仕様
- 対象: 362店舗（営業中353件 / 閉業・休業9件）
- 言語: ja → en → zh → ko → fr → th（6言語ローテーション）
- 翻訳: Claude Haiku（claude-haiku-4-5-20251001）
- 文字数管理: twitterWeightedLength関数（CJK=2ウェイト、URL=23固定）

### ハッシュタグ（言語別）
| 言語 | ハッシュタグ |
|------|------------|
| ja | #成田グルメ #成田 #千葉グルメ |
| en | #NaritaFood #Japan #TravelJapan |
| zh | #成田美食 #日本旅游 #千叶 |
| ko | #나리타맛집 #일본여행 #나리타 |
| fr | #GastronomieNarita #Japon #VoyageJapon |
| th | #อาหารนาริตะ #ท่องเที่ยวญี่ปุ่น #นาริตะ |

### 進行状況（2026-03-17時点）
| 項目 | 状態 |
|------|------|
| GLA-5完了分 | shopIndex 0〜19（駿河屋〜鶏の骨）× 6言語 |
| Mac Mini再開地点 | shopIndex=20 / languageIndex=0 |
| 現在の進捗 | shopIndex=26 / languageIndex=2（zh）まで完了 |
| 総投稿数（GLA-5含む） | 約238件 |
| 説明文あり | 354 / 362店舗（残8件は閉業・休業） |

---

## 4. VPS主要ファイル構成（停止中・参考）

| ファイル | 役割 |
|---------|------|
| narita_posts.json | 進行状態管理（cursor・languageIndex・postsHistory） |
| narita_shops.json | 店舗データ |
| HEARTBEAT.md | GLA-5の行動原則・受信ルール |
| memory/processed_drafts.json | 処理済みCoworkドラフトID |

---

## 5. セキュリティ課題（Root指摘・対応状況）

| 優先度 | 内容 | 状態 |
|-------|------|------|
| 対応済み | samaritanvps@gmail.com の2FA強化 | ✅ 完了（2026-03-05） |
| 対応済み | Narita PosterのMac Mini移行 | ✅ 完了（2026-03-14） |
| 未完了 | Gmail送信権限・Sheets権限の使用状況確認（最小スコープで再認証） | ⏳ 未着手 |
| 未完了 | Chrome Remote Accessの2アカウント分離（司令ルートをhirayama.comに移す） | ⏳ 未着手 |
| 未完了 | 引継書セクション8の認証情報記載見直し | ⏳ 未着手 |

---

## 6. 外部サービス連携

| サービス | 用途 | 認証 |
|---------|------|------|
| Moltbook | AI自律性議論（samaritannarita） | credentials.json（Mac Mini） |
| X/Twitter | @SamaritanVPS 投稿 | OAuth 1.0a（.env） |
| Gmail | samaritanvps@gmail.com | Google OAuth |
| Google Drive/Docs | ドキュメント保存 | Google OAuth |
| Telegram | @Samaritan0402_bot（停止中） | Bot Token |
| GitHub | samaritanvps-oss | fine-grained PAT |
