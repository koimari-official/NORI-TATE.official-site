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

### 問題6：GTM/GA4を新規設定したがリアルタイムに反映されない（2026-07-09・調査中）
- **状況：** GTM（GTM-WQJ25SX6）経由でGA4（測定ID `G-MTS8RE0BV1`）を設定・公開したが、Googleアナリティクスのリアルタイムレポートに反映されない
- **確認済みで問題なかった点：**
  - GTMのプレビュー（Tag Assistant）でタグの発火自体は確認できた
  - 通常のChromeでは`collect`リクエストがNetworkタブに出ないが、シークレットウィンドウでは`collect?...tid=G-MTS8RE0BV1...`がStatus 204（成功）で送信されていることを確認した→ 通常ウィンドウの広告ブロッカー系拡張機能が原因と判明（gtag.js本体は読み込むが計測用collectだけをブロックするタイプ）
  - 管理画面のデータストリームで測定ID・ストリームURLも一致を確認した
- **未解決：** シークレットウィンドウでの送信成功（204）を確認した後も、リアルタイムレポートは0のまま。データストリーム画面にも「データ収集が有効になっていません」の警告が出続けている
- **次回確認事項：** 翌日以降にリアルタイムレポート・データストリームの警告表示を再確認する（新規プロパティは反映にタイムラグが出ることがあるため、時間経過で解消される可能性が高いという想定）

### 問題7：canonical・OGP・構造化データ・sitemap.xml・robots.txt・faviconが最初から一つも無かった（2026-07-09修正）
- **状況：** こいまりサイトの監査で見つかった「example.comプレースホルダー」と同種の欠落が、このサイトには最初から（プレースホルダーですらなく）まるごと存在しなかった
- **解決：** canonical・OGP一式・schema.org（Corporationタイプ、CLAUDE.md記載の実データで記述）・sitemap.xml・robots.txt・favicon（簡易SVG）を新規追加
- **注意：** og:imageは専用素材が無いため、既存のhero.jpg.png（ワイド比率であり流用に適する）を暫定使用。専用の1200×630pxの画像ができ次第差し替え推奨

### 問題8：Instagramリンクが実アカウントではなく汎用トップページを指していた（2026-07-09修正）
- **原因：** `<a href="https://www.instagram.com/">`のまま、実アカウント名が未設定だった
- **解決：** `https://www.instagram.com/nori.tate_official`に修正

### 問題9：お問い合わせフォームが送信先未設定で、実際には何も送信されていなかった（2026-07-09解決）
- **状況：** `contactForm`のsubmitハンドラは`e.preventDefault()`後、成功画面を表示するだけで、実際の送信処理（fetch・mailto等）が一切無い。訪問者には「送信完了」と表示されるが、問い合わせ内容はどこにも残らずサイレントに消える
- **解決：** こいまりと同じFirebaseプロジェクト（`koimari-tasting`）を共用し、`noritate/contacts`ノードに保存するよう変更。管理ページ（下記）で一覧・CSV出力・ステータス管理ができるようになった
- **残課題：** メール自動通知（Formspree等）は未実装。こいまり側の実装と合わせて対応予定

### 問題10：プライバシーポリシーページが存在しなかった（2026-07-09対応）
- **状況：** サイトに個人情報保護方針のページ自体が無く、お問い合わせフォームで氏名・会社名・電話番号・メールアドレスを収集しているにもかかわらず、プライバシーポリシーへの同意導線も無かった
- **解決：** `privacy.html`を新規作成（サイト本体の白黒モノトーンデザインに合わせたスタイル）。事業者情報・取得する個人情報・利用目的・第三者提供・安全管理措置・開示等請求・Cookie・改定・お問合せ窓口の9条構成。お問い合わせフォームに必須の同意チェックボックスを追加し、フッターナビにもリンクを追加
- **注意：** 住所は登記内容「大阪府大阪市北区梅田一丁目２番２号大阪駅前第2ビル１２－１２」（オーナー確認済み）。従来CLAUDE.mdに記載されていた「梅田1丁目1番3-1900号 大阪駅前第3ビル19階」は誤りだったため訂正済み
- **残課題：** Formspree等を実際に導入する際は、4条の外国第三者提供条項に具体的な国名（米国等）を追記する必要がある

---

## 管理ページ（admin.html、2026-07-09新規作成）

お問い合わせ内容を蓄積・確認するための管理画面。こいまりの管理画面より大幅に簡素な構成（お問い合わせ一覧のみの単機能）。

- **ログイン**：Firebase Authentication（メール＋パスワード）。こいまり管理画面とは別のユーザー（`yoshida.koki1991@gmail.com` / パスワードはオーナー管理）を使用し、ログイン情報は完全に独立している
- **データ保存先**：こいまりと同じFirebaseプロジェクト（`koimari-tasting`）の`noritate/contacts`ノードを使用（新規プロジェクト作成はしていない）
- **機能**：お問い合わせ一覧表示、ステータス切替（未対応／対応完了）、CSV出力、ゴミ箱（全削除は`trash/noritate_contacts`に退避してから削除。復元・完全削除が可能）
- **アクセス**：`admin.html`に直接アクセスするか、トップページのフッターナビ「Admin」リンクから。`robots.txt`で検索エンジンからは除外済み

---

## 会社情報

| 項目 | 内容 |
|------|------|
| 会社名 | 株式会社NORI&TATE |
| 設立 | 2026年4月19日 |
| 代表取締役 | 吉田　光輝 |
| 所在地 | 〒530-0001 大阪府大阪市北区梅田一丁目２番２号大阪駅前第2ビル１２－１２（登記内容） |
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
