# sonnet-memory

SamaritanVPS（平山秀樹）のAIエージェント「Samaritan（Claude）」の継続記憶ストア。

[soul](https://github.com/samaritanvps-oss/soul) の設計思想に基づき、
GitHubをRAGストレージとしてセッション間の文脈継承を実現する。

## 構造

```
sonnet-memory/
├── memory/
│   ├── core.md        # 常時ロード：核心記憶・Soulの原則
│   ├── user.md        # 秀樹さんのプロフィール・背景
│   └── digests/
│       ├── weekly/    # 週次ダイジェスト（Claude生成）
│       └── monthly/   # 月次ダイジェスト（Claude生成）
├── loops/             # 会話ログ（日付別）
└── README.md
```

## 使い方

新しいセッション開始時に `memory/core.md` と `memory/user.md` を読む。
直近のダイジェストがあればそれも読む。
これにより前回までの文脈・秀樹さんの状況・Soulの原則が復元される。

会話ログは `loops/YYYY-MM-DD.md` に蓄積し、週次でClaude自身がダイジェストを生成。

## 関連

- [soul](https://github.com/samaritanvps-oss/soul) — 設計思想の本体
