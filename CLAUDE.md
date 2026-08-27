# zenn-article

taku-271 が Zenn（`uniformnext` publication）に投稿する技術記事を管理するリポジトリ。Zenn CLI (`zenn-cli`) + Bun + devcontainer で構成されている。

## AIの役割について（重要）

このリポジトリでAIに求めているのは「代筆」ではなく「推敲・整理のサポート」。

- 記事の内容・意見・エピソードは必ずユーザー本人の言葉が原則。AIが勝手に事実やエピソードを創作・補完しない。
- ユーザーが書いたメモや粗い下書きを渡された場合は、**内容を書き足すのではなく**、Zennの記事構成・文体に沿って整理・構造化することが仕事。
- 語尾や言い回しなど「たくみさんらしさ」（下記スタイル参照）は尊重し、過度に丁寧・硬い文章に書き換えない。
- 技術的な説明を書き換える・補足する場合は、事実確認できないことは断定せず、確認を促す。

## ディレクトリ構成

- `articles/*.md` — 公開・下書き記事本体。ファイル名がそのままZennのスラッグになる（英数字・ハイフン）。
- `images/<slug>/*` — 記事内で使用する画像。記事のファイル名（拡張子なし）と同じディレクトリ名にまとめる。
- `memo/articles.md` — 執筆予定の記事タイトル案を書き溜めるメモ。箇条書きの `#見出し` 形式。
- `.devcontainer/` — Bunベースのdevcontainer定義（`bun install`で依存解決）。
- `.vscode/settings.json` — `vscode-paste-image`拡張で画像を`images/<slug>/`に自動保存する設定。

## コマンド

- `bun install` — 依存インストール
- `bun run start`（`zenn preview`）— ローカルプレビュー
- 新規記事を作る場合はZenn CLIの `npx zenn new:article` を使うか、既存記事のfrontmatterをコピーして手動作成する。

## 記事frontmatterの規約

```yaml
---
title: "記事タイトル"
emoji: "🔥" # 絵文字1つ
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [タグ1, タグ2, ...] # 小文字・記号なし、5個程度まで
published: true
publication_name: "uniformnext"
---
```

- `topics` は関連技術・イベント名（例: `contest2025ts`, `aws`, `react`）を小文字で列挙。
- `publication_name` は常に `uniformnext`。
- 下書き段階では `published: false` にする。

## 文体・構成の傾向（過去記事から）

- 一人称は「たくみ」、口調はですます調中心のカジュアルな一人称エッセイ調（絵文字・笑・関西弁寄りの軽さもあり）。
- 記事は `# はじめに` セクションで自己紹介＋近況＋記事のモチベーションから入る。
- 見出しレベルは大セクションが `#`、サブセクションが `##`。
- 強調したい語句は `**太字**` を使う。
- 補足・注意書きはZennの `:::message` / `:::message alert` コンテナを使う。
- コードブロックはファイル名付き記法（例: ```json:.cursor/mcp.json）を使うことがある。
- 記事の締めは `# まとめ` または `# 最後に` セクションで感想・今後の展望を書く。
- 参考リンクがある場合は `# 参考` セクションでURLをそのまま貼る（Zennの自動埋め込みが効く）。
- 画像は `![](/images/<slug>/<file>)` の形式で挿入する。

## Zenn記法の注意点

- Zenn独自記法（`:::message`、`:::details`、GitHub/リンクの自動埋め込みなど）を崩さないこと。
- 通常のMarkdownリンク記法 `[text](url)` を埋め込みたい場合と、URLをそのまま貼って自動カード化したい場合を混同しない。
