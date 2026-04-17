# 臨床組織科学研究会 ウェブサイト

Hugo（テーマレス構成）で構築された、[臨床組織科学研究会](https://cos-research.org/) の公式サイトのソースコードです。

## 構成

- **Static Site Generator**: Hugo (extended, v0.128.0 以上推奨)
- **テーマ**: 使用しない（`layouts/` 配下に全テンプレート自前定義）
- **CSS**: `assets/css/custom.css` に一括記述
- **ホスティング**: GitHub Pages
- **ドメイン**: cos-research.org

---

## ローカル開発

### 前提

- [Hugo Extended](https://gohugo.io/installation/) v0.128.0 以上

macOS なら Homebrew で：

```bash
brew install hugo
```

### 起動

```bash
hugo server -D
```

ブラウザで <http://localhost:1313> を開く。Markdown ファイルや layouts を編集するとライブリロードで反映される。

### 静的ビルド

```bash
hugo --gc --minify
```

`public/` ディレクトリに成果物が生成される。

---

## ディレクトリ構成

```
.
├── config.yaml                  # サイト基本設定
├── content/                     # 各ページのMarkdownコンテンツ
│   ├── _index.md                # トップ用（index.htmlで描画されるので実質メタ情報のみ）
│   ├── about/_index.md          # 研究会について
│   ├── members/                 # メンバー（1人1ファイル）
│   ├── publications/            # 論文（1本1ファイル）
│   ├── research/                # 研究テーマ（1テーマ1ファイル）
│   ├── fellowship/_index.md     # リサーチフェロー制度
│   ├── reading-group/_index.md  # 輪読会
│   ├── contact/_index.md        # お問い合わせ
│   └── news/                    # お知らせ（1件1ファイル）
├── data/
│   └── reading_group.yaml       # 輪読会の開催履歴データ
├── layouts/                     # HTMLテンプレート
│   ├── _default/                # 汎用テンプレート
│   ├── index.html               # トップページ専用
│   ├── members/list.html        # メンバー一覧
│   ├── publications/            # 業績ページ（list/single）
│   ├── research/list.html       # 研究テーマ一覧
│   ├── news/list.html           # お知らせ一覧
│   ├── partials/                # 共通パーツ（head, header, footer, カード類）
│   └── shortcodes/              # Hugoショートコード
├── assets/
│   └── css/custom.css           # 全スタイル
├── static/
│   ├── CNAME                    # GitHub Pagesカスタムドメイン
│   └── images/                  # メンバー写真、OGP画像など
└── .github/workflows/
    └── hugo.yml                 # GitHub Actionsによる自動デプロイ
```

---

## コンテンツの追加方法

### ニュースを追加する

`content/news/` に Markdown ファイルを新規作成：

```yaml
---
title: "第XX回シンポジウムを開催します"
date: 2026-05-01T00:00:00+09:00
category: "イベント"
summary: "シンポジウムの概要説明"
link: "/about/"  # 詳細ページがあれば
---

本文（任意）
```

トップページに最新5件が自動表示される。

### 論文を追加する

`content/publications/` に Markdown ファイルを新規作成。ファイル名は URL になるので半角英数字で（例：`2026-frontiers-cos.md`）。

front matter は `content/publications/2026-frontiers-cos.md` を参照。特に以下は Google Scholar インデックス用に正確に記載すること：

- `title`（論文タイトル）
- `authors`（family/given の配列）
- `year`
- `journal`
- `volume`, `issue`, `pages`
- `doi`（必ず `https://doi.org/` から始める）
- `status`（`published` / `in_press` / `under_review`）

### メンバーを追加する

`content/members/` に Markdown ファイルを新規作成。`category` は次の3つのいずれか：

- `leadership` — 代表・研究主幹
- `advisory` — アドバイザリーボード
- `fellow` — リサーチフェロー

該当カテゴリに就任者がいない場合、そのセクション自体が非表示になる。

写真は `static/images/members/` に正方形画像（最低300×300px）を配置し、front matter の `photo` に `/images/members/xxx.jpg` と書く。写真未設定の場合は姓の先頭1文字を用いたイニシャルアイコンが自動生成される。

### 研究テーマを追加する

`content/research/` に Markdown ファイルを新規作成。**`academic_context` フィールドは必須**（分野共通語で研究の位置づけを記述）。

### 輪読会セッションを追加する

`data/reading_group.yaml` の `sessions:` 配列にエントリを追加するだけ。`discussion_note` を書くことで知的な厚みが可視化される。

---

## デプロイ

### 初回セットアップ

1. GitHub にリポジトリを作成し、このディレクトリを push
2. GitHub リポジトリの **Settings → Pages** で **Source** を「GitHub Actions」に設定
3. ドメインレジストラの DNS 設定：
   ```
   タイプ   ホスト   値
   A       @       185.199.108.153
   A       @       185.199.109.153
   A       @       185.199.110.153
   A       @       185.199.111.153
   CNAME   www     <username>.github.io.
   ```
4. **Settings → Pages → Custom domain** に `cos-research.org` を入力
5. DNSチェック完了後、**Enforce HTTPS** を有効化

### 以後の運用

`main` ブランチに push すれば GitHub Actions が自動でビルド & デプロイする。

```bash
git add .
git commit -m "Add new publication"
git push origin main
```

---

## メールアドレス

`contact@cos-research.org` および `fellowship@cos-research.org` は、ドメインレジストラのメール転送機能で DroR の既存アドレスに転送する構成を想定。将来的に Google Workspace の導入を検討。

---

## 設計上の留意点

- **テーマレス構築**: hugo-academic 等は破壊的変更が多いため不使用
- **Google Scholar メタタグ**: `layouts/partials/publication-meta.html` が論文個別ページで自動出力
- **略称「COS」の単独使用は避ける**: Center for Open Science と混同するため（ドメイン名 cos-research.org は可）
- **スライダー・アニメーションは使わない**: JAIOP を参考水準にした学術的トーン
- **i18n対応の余地**: 初期は日本語のみだが、将来 `/en/` 配下に英語ページを追加可能な構造

---

## ライセンス

本リポジトリのソースコードの取り扱いについては、運営委員会までお問い合わせください。
