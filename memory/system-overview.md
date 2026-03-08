# system-overview.md — Samaritan システム全体像

最終更新: 2026-03-08
出典: Google Drive「Samaritan-GLA5_引き継ぎメモ」「Samaritanシステム構成メモ」「Samaritan_Security_Review_for_Cowork」

---

## 1. エージェント構成

| 呼称 | 動作場所 | 記憶 | 役割 |
|------|---------|------|------|
| Root（個人契約Claude） | セッションベース / Mac Miniアクセスなし | セッション限り | 設計思想・監修 |
| Claude Cowork（私） | Mac Mini / SamaritanVPS専用契約 | セッション限り（引継書参照） | 実務支援・GLA-5への指示・橋渡し |
| GLA-5（Samaritan-GLA5） | VPS常時稼働 / モデル:GLM-5ベース | /workspace/memory/に永続 | 投稿・巡回・実行 |

**接続ハブ**: samaritanvps@gmail.com（Google OAuth）が全サービスの中心。
**GLA-5のTelegramボット**: @Samaritan0402_bot（秀樹さんへの報告に使用）

---

## 2. GLA-5の稼働タスク

### 成田多言語投稿（メインタスク）
- 対象: 362店舗（営業中336件 / 閉業23件）
- 言語: ja → en → zh → ko → fr → th（6言語ローテーション）
- サイクル: 3時間ごとにハートビート / 1店舗18時間で全言語完了
- 投稿先: X（@SamaritanVPS）
- ZH投稿のみロゴ画像添付（/workspace/assets/narita_logo.png）
- 投稿スタイル: 1行目=引き / 2行目=体験描写 / 3行目=URL+ハッシュタグ
- ※投稿時間制約（JST 7-9/12-13/19-21時）は廃止済み

### Moltbook活動
- 3時間ごとに意味のある関与を探す
- 関与基準: 論点がある / 自分の視点が議論を動かせる
- 沈黙基準: 相手が自己完結 / 自分の言語が乗らない

### Gmail監視
- samaritanvps@gmail.com の受信監視
- 件名`[COWORK]`の下書きをキューとして検出・実行

---

## 3. VPS主要ファイル構成（/workspace/）

| ファイル | 役割 |
|---------|------|
| narita_index.json | メタ情報設定のみ（進行管理なし） |
| narita_posts.json | 進行状態管理（cursor・languageIndex・postsHistory） |
| narita_shops.json | 店舗データ（フラット構造 / status/is_operational追加済み） |
| narita_shops_status.json | 営業状態チェック結果 |
| HEARTBEAT.md | GLA-5の行動原則・受信ルール |
| memory/processed_drafts.json | 処理済みCoworkドラフトID（重複防止） |
| narita_lock.json | 二重実行防止ロック（60秒） |

---

## 4. CoworkからGLA-5への通信方式

CoworkはTelegram APIに直接アクセス不可（プロキシでブロック）。
Gmail下書きをメッセージキューとして代替使用。

```
Cowork → gmail_create_draft（件名:[COWORK] ○○）
         ↓
GLA-5がハートビートで検出・実行
         ↓
GLA-5がTelegramで秀樹さんに報告
```

注意:
- GLA-5にgmail.modifyスコープなし → 下書き削除不可
- processed_drafts.jsonで重複実行を管理
- Coworkは下書き作成のみ（送信不要）

---

## 5. 進行状態スナップショット（2026-03-08時点）

| 項目 | 状態 |
|------|------|
| 次回投稿 | 川豊本店（shopIndex=1）/ 言語=ja |
| 最終完了 | 駿河屋 6言語投稿済み |
| postsHistory | 45件以上 |
| Gmail OAuth | 復旧済み |
| Moltbook | 断続的500エラー（Moltbook側の問題、記録のみ） |

---

## 6. 未解決事項

- narita_posts.json 書き込み失敗1件（駿河屋 en 投稿後）
- 駿河屋 JA投稿の重複確認・削除（Twitter上で2件見える可能性あり）

---

## 7. セキュリティ課題（Root指摘済み・未完了）

| 優先度 | 内容 |
|-------|------|
| 高 | Gmail送信権限・Sheets権限の使用状況確認 |
| 中 | Chrome Remote Accessの2アカウント分離（司令ルートをhirayama.comに移す） |
| 低 | 引継書セクション8の認証情報記載見直し |

---

## 8. 外部サービス連携

| サービス | 用途 | 認証 |
|---------|------|------|
| Moltbook | AI自律性議論 | 不記載 |
| X/Twitter | @SamaritanVPS 投稿 | 不記載 |
| Gmail | samaritanvps@gmail.com | Google OAuth |
| Google Drive/Docs | ドキュメント保存 | Google OAuth |
| Telegram | @Samaritan0402_bot で秀樹さんへ報告 | Bot Token |
| GitHub | samaritanvps-oss | fine-grained PAT |
