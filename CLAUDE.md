# CLAUDE.md — 株式会社NORI&TATE サイト 実装詳細

> このファイルはNORI&TATEサイト固有の実装仕様です。  
> クロスデバイス対応の汎用ルールは `MANUAL.md` を参照してください。

---

## プロジェクト概要

株式会社NORI&TATEの企業ホームページ。静的HTML単一ファイル構成。  
再生成時は `index.html` 1ファイルを再構築すればサイト全体が復元できる。

## 技術スタック

- 静的HTML（`index.html` 1ファイル）
- Google Fonts（Noto Serif JP / Noto Sans JP / Montserrat）
- バニラCSS（外部ライブラリなし）
- バニラJS（最小限・IntersectionObserver）
- アクセス解析: Google Tag Manager（ID: `GTM-WQJ25SX6`、2026-07-09導入。head直後・body直後にスニペット設置済み）

---

## レイアウト構造（三列構成）

### ヒーロー表示中（`body.hero-mode`）
- サイドバー・右パネルは非表示（`opacity: 0`・`pointer-events: none`）
- `#main` のマージンは 0（全画面表示）
- `hero.jpg.png` が全画面に表示され、中央に大見出しを配置

### スクロール後（通常モード）
- **左サイドバー**（`#sidebar`、240px固定、白背景）：ロゴ＋縦ナビ
- **中央エリア**（`#main`、margin-left: 240px）：スクロール
- **右サイドバー**（`#right-sidebar`、180px固定、透明）：SNS・連絡先を下部に表示
- ハンバーガーメニューは使用しない

### 背景レイヤー（`#bg-layer`）
- `z-index: 1`、`position: fixed`、`height: 100lvh`（iOSズーム防止）
- `active-hero`（`opacity: 1`）：Heroセクション（hero.jpg.png を鮮明に表示）
- `active-light`（`opacity: 0.12`）：各コンテンツセクション（半透明）
- セクションに入るたびにIntersectionObserverで切替

---

## ファイル構成

```
/
├── index.html          ← メインページ（全セクション含む）
├── hero.jpg.png        ← トップ画像
├── our mind.jpg.png    ← Conceptセクション背景
├── ameft.jpg.jpg       ← Nameセクション背景
├── cake.jpg.jpg        ← Businessセクション背景
├── CLAUDE.md           ← このファイル
└── MANUAL.md           ← 汎用ガイド
```

---

## セクション構成

| セクションID | 表示名 | 内容 | 背景パネルID |
|-------------|--------|------|------------|
| `#top` | Top | ヒーロー全画面 | `bg-top`（hero.jpg.png） |
| `#concept` | Concept | 私たちの考え方 | `bg-concept`（our mind.jpg.png） |
| `#name` | Name | 社名の由来 | `bg-name`（ameft.jpg.jpg） |
| `#business` | Business | 事業実績（こいまり） | `bg-business`（cake.jpg.jpg） |
| `#growth` | Key Principles | 事業成長の心得 | `bg-growth`（Unsplash） |
| `#join` | Join | オーナー様へ（**黒背景**） | `bg-growth`（Unsplashを共用） |
| `#company` | Company | 会社概要 | `bg-company`（Unsplash） |
| `#contact` | Contact | お問い合わせフォーム | `bg-contact`（Unsplash） |

---

## CSS変数（`:root`）

```css
:root {
  --serif:  'Noto Serif JP', serif;
  --sans:   'Noto Sans JP', sans-serif;
  --en:     'Montserrat', sans-serif;
  --white:  #ffffff;
  --black:  #111111;
  --mid:    #888888;
  --border: #e8e8e8;
  --col-l:  240px;   /* 左サイドバー幅 */
  --col-r:  180px;   /* 右サイドバー幅 */
}
```

---

## レスポンシブブレークポイント

| 幅 | 条件 | 適用内容 |
|----|------|---------|
| 960px超 | デフォルト | 三列・右サイドバー表示・`.sec-head` sticky |
| 960px以下 | `max-width: 960px` | 右サイドバー非表示、`.sec-head` static、単列 sec-split |
| 660px以下 | `max-width: 660px` | 左サイドバー→上部固定ナビ。`--col-l: 0` |
| 横向きスマホ | `orientation: landscape` + `max-height: 500px` | サイドバーpadding縮小・sec-split padding縮小 |

---

## セクションレイアウト（`.sec-split`）

```css
.sec-split {
  display: grid;
  grid-template-columns: 200px 1fr;  /* 左：表題、右：本文 */
  gap: 4.5rem;
  padding: 8rem 4.5rem;
}
.sec-head { position: sticky; top: 5rem; }
/* ↑ 960px以下では position: static に上書き */
```

---

## JS実装（ヒーローモード＋セクション検知）

```javascript
// ── 背景切替 ──
const bgMap = {
  top:      { panel:'bg-top',      mode:'hero'  },
  concept:  { panel:'bg-concept',  mode:'light' },
  name:     { panel:'bg-name',     mode:'light' },
  business: { panel:'bg-business', mode:'light' },
  growth:   { panel:'bg-growth',   mode:'light' },
  join:     { panel:'bg-growth',   mode:'light' },  // growthと共用
  company:  { panel:'bg-company',  mode:'light' },
  contact:  { panel:'bg-contact',  mode:'light' },
};

// ── ヒーローモード（再初期化対応） ──
let _heroObs = null;
function initHeroObserver() {
  if (_heroObs) _heroObs.disconnect();
  const threshold = window.matchMedia('(max-width: 660px)').matches ? 0.05 : 0.7;
  _heroObs = new IntersectionObserver((entries) => {
    document.body.classList.toggle('hero-mode', entries[0].isIntersecting);
  }, { threshold });
  _heroObs.observe(document.getElementById('top'));
}

// ── セクション検知（再初期化対応） ──
let _sectionObs = [];
function initSectionObservers() {
  _sectionObs.forEach(o => o.disconnect());
  _sectionObs = [];
  const navH = window.matchMedia('(max-width: 660px)').matches ? 70 : 0;
  const mid  = Math.round((window.innerHeight - navH) / 2);
  const topM = navH + mid;
  const botM = Math.max(0, window.innerHeight - topM);
  const rootMargin = `-${topM}px 0px -${botM}px 0px`;
  document.querySelectorAll('section[id]').forEach(s => {
    const obs = new IntersectionObserver((entries) => {
      entries.forEach(e => {
        if (e.isIntersecting) { activateBg(e.target.id); setNav(e.target.id); }
      });
    }, { threshold: 0, rootMargin });
    obs.observe(s);
    _sectionObs.push(obs);
  });
}

// ── 画面回転・リサイズ時に再初期化（250ms デバウンス） ──
let _resizeTimer;
window.addEventListener('resize', function() {
  clearTimeout(_resizeTimer);
  _resizeTimer = setTimeout(function() {
    initHeroObserver();
    initSectionObservers();
  }, 250);
});
```

---

## このサイトで発生した問題と解決策

### 問題1：スマホで白背景がページ全体を覆う
- **原因：** `#sidebar { bottom: 0; }` がモバイルでリセットされていなかった
- **解決：** `@media (max-width: 660px)` 内に `bottom: auto; height: auto;` を追加

### 問題2：スマホ横向きで背景画像・ナビ同期が停止する
- **原因：** Observerの `rootMargin` が縦向き時の `innerHeight` で固定されたまま横向きに適用された（縦向き `topM` が横向き `innerHeight` を超えて検知不能に）
- **解決：** Observer を関数化し `resize` イベントで再初期化

### 問題3：iOSで背景がスクロール時にズームする
- **原因：** `position: fixed; inset: 0` の背景レイヤーがアドレスバーの出入りによる `100vh` 変動でリサイズされる
- **解決：** `height: 100lvh` に変更（`100vh` はフォールバックとして残す）

### 問題4：横向きスマホでナビ下部（Company・Contact）が見切れる
- **原因：** サイドバーの `padding-top: 3.5rem` が高さ375px程度の横向き画面に収まらない
- **解決：** `@media (orientation: landscape) and (max-height: 500px)` でpadding縮小・`overflow-y: auto`

### 問題5：Nameセクション見出しの `(ノリタテ)` が折り返す
- **原因：** 表題列200pxに対し、ブラウザのフォントサイズ設定によっては文字幅が超過
- **解決：** `#name .s-title { white-space: nowrap; }` を追加

---

## 会社情報

| 項目 | 内容 |
|------|------|
| 会社名 | 株式会社NORI&TATE |
| 設立 | 2026年4月19日 |
| 代表取締役 | 吉田　光輝 |
| 所在地 | 大阪市北区梅田1丁目1番3-1900号 大阪駅前第3ビル19階 |
| 電話 | 090-1243-6770 |
| メール | noritate.official@gmail.com |
| Instagram | https://www.instagram.com/nori.tate_official |

---

## 修正時の注意

- 背景画像の差し替え：`#bg-layer` 内の各 `.bg-panel` の `style="background-image:url(...)"` を変更
- ヒーロー画像ファイル名は `hero.jpg.png`（拡張子に注意）
- `#join` セクションのみ `background: var(--black)` を持つ。他セクションは背景なし（透明）
- 新しいセクションを追加する場合は `bgMap` にも追加すること
- Observer を追加・変更する場合は必ず `initSectionObservers()` 内に記述し、`resize` での再初期化に含めること
