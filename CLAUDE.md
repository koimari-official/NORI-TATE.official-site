# CLAUDE.md — 株式会社NORI&TATE サイト 作業ルール

## プロジェクト概要

株式会社NORI&TATEの企業ホームページ。静的HTML単一ファイル構成。

## 技術スタック

- 静的HTML（index.html 1ファイル）
- Google Fonts（Noto Serif JP / Noto Sans JP / Montserrat）
- バニラCSS（外部ライブラリなし）
- バニラJS（最小限・IntersectionObserver）

## レイアウト構造（三列構成）

### ヒーロー表示中（body.hero-mode）
- サイドバー・右パネルは非表示（opacity: 0）
- `#main` のマージンは 0（全画面表示）
- `hero.jpg.png` が全画面に表示され、中央に大見出しを配置

### スクロール後（通常モード）
- **左サイドバー**（`#sidebar`、240px固定、白背景）：ロゴ＋縦ナビ
- **中央エリア**（`#main`、margin-left: 240px / margin-right: 220px）：スクロール
- **右サイドバー**（`#right-sidebar`、220px固定、透明）：SNS・連絡先を下部に表示
- ハンバーガーメニューは使用しない

### 背景レイヤー（`#bg-layer`、z-index: 1）
- `active-hero`（opacity: 1）：Heroセクション（hero.jpg.png を鮮明に表示）
- `active-light`（opacity: 0.12）：各コンテンツセクション（半透明・白が透けて見える）
- セクションに入るたびにIntersectionObserverで切替

## ファイル構成

```
/
├── index.html      ← メインページ（全セクション含む）
├── hero.jpg.png    ← トップ画像（ユーザーが配置）
├── CLAUDE.md       ← このファイル
└── MANUAL.md       ← テンプレート活用手順書
```

## セクション構成

1. `#bg-layer`（固定背景レイヤー、z-index: 1）
2. `#sidebar`（左固定ナビ、z-index: 30、白背景）
3. `#right-sidebar`（右固定パネル、z-index: 20、透明）
4. `#main`（中央スクロールエリア、z-index: 10）
   - `#top`（Hero：全画面、中央揃え、`body.hero-mode` 時にサイドバー非表示）
   - `#concept`（私たちの考え方）
   - `#name`（社名の由来）
   - `#business`（事業実績：こいまり）
   - `#growth`（事業成長の心得）
   - `#join`（For Owners：**黒背景**のみ）
   - `#company`（会社概要）
   - `#contact`（お問い合わせフォーム）
   - `#footer`

## CSS変数（`:root`）

| 変数 | 値 | 用途 |
|------|-----|------|
| `--col-l` | 240px | 左サイドバー幅 |
| `--col-r` | 220px | 右サイドバー幅 |
| `--serif` | Noto Serif JP | 見出し・本文明朝 |
| `--sans`  | Noto Sans JP  | ナビ・UI系ゴシック |
| `--en`    | Montserrat    | 英字ラベル・ボタン |

## セクションレイアウト（`.sec-split`）

各コンテンツセクションは二列グリッド：

```css
.sec-split {
  display: grid;
  grid-template-columns: 200px 1fr;  /* 左：表題、右：本文 */
  gap: 4.5rem;
  padding: 8rem 4.5rem;
}
.sec-head { position: sticky; top: 5rem; }  /* 表題は左に固定 */
```

## ヒーローモード制御（JS）

```javascript
// ヒーローが8%以上見えている間は hero-mode ON
new IntersectionObserver((entries) => {
  document.body.classList.toggle('hero-mode', entries[0].isIntersecting);
}, { threshold: 0.08 }).observe(document.getElementById('top'));
```

## フェードインアニメーション

- クラス `.js-fade`：opacity 0 + translateY(36px) → `.on` クラスで解除
- 遅延クラス：`.d1`（0.15s）/ `.d2`（0.28s）/ `.d3`（0.40s）/ `.d4`（0.52s）
- IntersectionObserver（threshold: 0.1）で一度だけ発火

## レスポンシブブレークポイント

| ブレークポイント | 条件 | 適用内容 |
|----------------|------|---------|
| PC | 幅 960px超 | 三列レイアウト。右サイドバー表示。`.sec-head` sticky |
| タブレット〜横向きスマホ | 幅 661〜960px | 右サイドバー非表示。`.sec-head` static。単一列 sec-split |
| 縦向きスマホ | 幅 660px以下 | 左サイドバーが上部固定ナビに変化。`--col-l: 0` |
| 横向きスマホ（追加） | `orientation: landscape` かつ 高さ 500px以下 | サイドバーパディング縮小・sec-split padding縮小 |

## IntersectionObserver の扱い（重要）

- `IntersectionObserver` の `rootMargin` は初期化時の `window.innerHeight` で計算される
- **画面回転・リサイズ後は古い rootMargin が無効になり、背景切替・ナビ同期が停止する**
- 対処：`initHeroObserver()` と `initSectionObservers()` を関数化し、`resize` イベント（250msデバウンス）で再初期化する
- 新たに Observer を追加・変更する際も必ず再初期化関数の中に記述すること

```javascript
// Observer再初期化パターン（必ず守る）
let _obs = [];
function initObservers() {
  _obs.forEach(o => o.disconnect());
  _obs = [];
  // ここで new IntersectionObserver(...) を作成し _obs.push(obs)
}
initObservers();
let _timer;
window.addEventListener('resize', function() {
  clearTimeout(_timer);
  _timer = setTimeout(initObservers, 250);
});
```

## スマホ横向き対応の注意点

- iOS でアドレスバーが出入りする際に `100vh` が変動し背景がズームする → `height: 100lvh` で防ぐ
- `#sidebar` の `bottom: 0` は縦向きモバイルCSSで `bottom: auto` にリセット必須
- 横向きスマホ（幅 > 660px）は縦向きの660pxメディアクエリが適用されない点に注意
- `window.matchMedia('(max-width: 660px)')` は横向きでは false になる

## クロスデバイス作業ルール

コードを変更したら以下の3環境を必ず想定してテストすること：

1. **PC（960px超）** — 三列・sticky表題・右サイドバーあり
2. **スマホ縦向き（660px以下）** — 上部固定ナビ・単列
3. **スマホ横向き（661〜959px・高さ≦500px）** — 左サイドバー・単列・Observer再初期化必要

## 修正時の注意

- 背景画像の差し替え：`#bg-layer` 内の各 `.bg-panel` の `style="background-image:url(...)"` を変更
- ヒーロー画像ファイル名は `hero.jpg.png`（拡張子に注意）
- `#join` セクションのみ `background: var(--black)` を持つ。他セクションは背景なし（透明）
- 右サイドバーのSNS URL（Instagram等）は `#right-sidebar` 内の `<a href="...">` を変更
- インラインスタイルより `<style>` タグ内のCSS変数を優先
- レスポンシブ：960px以下で右サイドバー非表示、660px以下でサイドバーが上部バーに変化
