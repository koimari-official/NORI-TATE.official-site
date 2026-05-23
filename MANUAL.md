# 株式会社NORI&TATE サイト 運用・カスタマイズ手順書

---

## 目次

1. [サイト概要・技術構成](#1-サイト概要技術構成)
2. [ファイル構成](#2-ファイル構成)
3. [画面構成・レイアウト](#3-画面構成レイアウト)
4. [よく使う更新作業](#4-よく使う更新作業)
5. [デザインのカスタマイズ](#5-デザインのカスタマイズ)
6. [GitHub Pagesへのデプロイ](#6-github-pagesへのデプロイ)

---

## 1. サイト概要・技術構成

| 項目 | 内容 |
|------|------|
| 構成 | 静的HTML（サーバー不要） |
| ファイル数 | index.html 1ファイル |
| 公開方法 | GitHub Pages（無料）または任意のWebサーバー |
| デザイン | バニラCSS（外部CSSライブラリなし） |
| フォント | Google Fonts（Noto Serif JP / Noto Sans JP / Montserrat） |
| インタラクション | バニラJS（IntersectionObserver によるフェードイン・背景切替） |

**メリット：** サーバー代不要、維持コストほぼゼロ、読み込みが速い

---

## 2. ファイル構成

```
/
├── index.html      ← メインページ（全コンテンツ含む）
├── hero.jpg.png    ← トップ画像（必須・この名前で配置）
├── CLAUDE.md       ← AI作業ルール（Claude Code用）
└── MANUAL.md       ← この手順書
```

> **注意：** ヒーロー画像のファイル名は必ず `hero.jpg.png` にしてください。
> 別の名前にする場合は `index.html` 内の `url('hero.jpg.png')` を変更してください。

---

## 3. 画面構成・レイアウト

### ヒーロー表示（最初の全画面）

ページを開くと `hero.jpg.png` が全画面表示され、中央に以下が表示されます：

- 英語サブタイトル（小文字・薄め）
- **メインキャッチコピー**（大見出し・明朝体）
- 社名「株式会社NORI&TATE」

サイドバーはこの時点では非表示です。

### スクロール後の三列レイアウト

ヒーローをスクロールすると自動的に三列レイアウトへ切り替わります：

| 列 | 幅 | 内容 |
|----|-----|------|
| 左サイドバー | 240px（固定） | ロゴ＋ナビゲーション |
| 中央エリア | 残りの幅（スクロール） | 各セクションの内容 |
| 右サイドバー | 220px（固定） | SNSアイコン・連絡先（下部） |

### セクション一覧

| セクションID | 表示名 | 内容 |
|-------------|--------|------|
| `#top` | Top | ヒーロー（全画面） |
| `#concept` | Concept | 私たちの考え方 |
| `#name` | Name | 社名の由来（NORI&TATE） |
| `#business` | Business | 事業実績（こいまり） |
| `#growth` | 心得 | 事業成長の心得（3つの柱） |
| `#join` | Join | オーナー様へ（黒背景） |
| `#company` | Company | 会社概要 |
| `#contact` | Contact | お問い合わせフォーム |

---

## 4. よく使う更新作業

### 4-1. ヒーロー画像を差し替える

新しい画像ファイルを `hero.jpg.png` という名前でフォルダに上書き保存するだけです。

別の名前にしたい場合は `index.html` 内を検索：
```
url('hero.jpg.png')
```
この部分を新しいファイル名に変更します。

### 4-2. キャッチコピーを変更する

`index.html` の Hero セクション（`<section id="top">` 内）を編集します：

```html
<span class="hero-en">Reviving Japan through Small Business</span>
<h1 class="hero-catch">中小企業から、<br>再び日本を<br>盛り上げる。</h1>
<p class="hero-brand">株式会社NORI&amp;TATE</p>
```

> `&amp;` は `&` の HTML 表記です。そのまま残してください。

### 4-3. 各セクションの本文を更新する

各セクションは以下の構造になっています：

```html
<section id="concept">
  <div class="sec-split">
    <div class="sec-head">  <!-- 左側の表題（スクロール中も固定） -->
      <span class="s-label">Concept</span>
      <h2 class="s-title">私たちの考え方</h2>
    </div>
    <div class="sec-body">  <!-- 右側の本文（ここを編集） -->
      <p class="s-body">本文テキスト</p>
    </div>
  </div>
</section>
```

### 4-4. 会社概要を更新する

`<section id="company">` 内のテーブルを編集します：

```html
<tr><th>会社名</th><td>株式会社NORI&amp;TATE</td></tr>
<tr><th>設立</th><td>2026年4月19日</td></tr>
<tr><th>代表取締役</th><td>吉田　光輝</td></tr>
<tr><th>所在地</th><td>大阪市北区梅田...</td></tr>
<tr><th>連絡先</th><td>090-1243-6770<br><a href="mailto:...">...</a></td></tr>
```

### 4-5. SNSリンク・連絡先を更新する

右サイドバー（`<div id="right-sidebar">` 内）を編集します：

```html
<!-- Instagram URL を変更 -->
<a href="https://www.instagram.com/あなたのアカウント" ...>

<!-- 電話番号 -->
<a href="tel:09012436770">090-1243-6770</a>

<!-- メールアドレス -->
<a href="mailto:noritate.official@gmail.com">noritate.official@gmail.com</a>
```

### 4-6. 事業実績（こいまり）を更新する

`<section id="business">` 内の `.biz-card` を編集します。
新しい事業を追加する場合はカードをコピーして `border-top` で区切ります。

---

## 5. デザインのカスタマイズ

### 5-1. カラーを変更する

`index.html` 先頭の `:root {}` ブロックを編集します：

```css
:root {
  --white:  #ffffff;   /* 背景・サイドバー */
  --black:  #111111;   /* テキスト・Join セクション背景 */
  --mid:    #888888;   /* サブテキスト・ラベル */
  --border: #e8e8e8;   /* 区切り線 */
  --col-l:  240px;     /* 左サイドバー幅 */
  --col-r:  220px;     /* 右サイドバー幅 */
}
```

### 5-2. ヒーロー文字サイズを変更する

```css
.hero-catch {
  font-size: clamp(最小, 画面幅比, 最大);
  /* 現在値：clamp(3.2rem, 7.5vw, 7rem) */
}

.hero-brand {
  font-size: clamp(1.1rem, 2.2vw, 1.6rem);
}
```

### 5-3. 背景画像（各セクション）を変更する

`#bg-layer` 内の各パネルの URL を変更します：

```html
<div class="bg-panel" id="bg-concept"
  style="background-image:url('あなたの画像URL');">
</div>
```

現在は Unsplash の画像を使用しています（インターネット接続が必要）。
ローカル画像を使う場合はファイルをフォルダに置いて `url('ファイル名.jpg')` に変更します。

### 5-4. 背景の透過度を変更する

```css
.bg-panel.active-hero  { opacity: 1; }    /* ヒーロー：鮮明 */
.bg-panel.active-light { opacity: 0.12; } /* 各セクション：半透明 */
```

`active-light` の値を上げると背景画像が濃くなります（0〜1の範囲）。

### 5-5. ヒーローモードの切替タイミングを変更する

```javascript
// threshold: 0.08 → ヒーローが8%以下になったらサイドバーを表示
}, { threshold: 0.08 }).observe(document.getElementById('top'));
```

値を大きくする（例：0.3）とヒーローが残っているうちにサイドバーが現れます。

---

## 6. GitHub Pagesへのデプロイ

### 初回設定

1. GitHub でリポジトリを作成（Public）
2. `index.html` と `hero.jpg.png` をリポジトリにアップロード
3. Settings → Pages → Source: `main` ブランチ → Save
4. 数分後に `https://USERNAME.github.io/REPO_NAME/` で公開

### 独自ドメインを使う場合

1. ドメインを取得（お名前.com、Xserverドメイン等）
2. DNS設定でGitHub PagesのIPアドレスに向ける
3. Settings → Pages → Custom domain に入力
4. HTTPSを有効化

### 更新の流れ

```bash
# ファイルを編集後
git add index.html
git commit -m "更新内容の説明"
git push origin main
# → 数分後に自動反映
```

---

*最終更新：2026年5月*
