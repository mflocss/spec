# mFLOCSS 仕様

---

## Abstract

*This section is informative.*

mFLOCSS は、CSS の記述を層に分類する思考フレームワークである。「このスタイルをどの層に書くか」という判断を体系化し、設計の一貫性と長期的な保守性を実現する。本仕様はその判断基準・命名規則・ファイル構成を厳密に定義する。

---

## Status of This Document

*This section is informative.*

本文書は mFLOCSS 仕様書のドラフト版である。v1.0 で正式版となる予定であり、v1.0 リリース以前は仕様の内容が変更される可能性がある。

**最終更新**: 2026-04-11  
**著者**: shunei

---

## 1. Overview

### 1.1 Introduction

*This section is informative.*

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークであり、ルールブックではない。

「どの層に、なぜ書くか」という問いに対し、明確な判断基準を提供する。`@layer` ベースのフラットアーキテクチャを採用する。`@layer`（CSS Cascading and Inheritance Level 5 [CSS-CASCADE-5]）は、スタイルの優先順位をセレクタ詳細度に依存せず宣言順で制御する CSS 標準機能である。

本仕様はその判断基準を厳密に定義する。

#### 対象範囲

- ビルドツール・フレームワーク非依存
- プリプロセッサを前提としない（併用可能）

### 1.2 不変原則

*This section is normative.*

mFLOCSS は以下の 4 原則に基づく。層数が変わってもこれらは変わらない。

1. **関心の分離** — 異なる問いに答えるスタイルは異なる層に分離する
2. **@layer による構造的制御** — 詳細度の問題を命名規則ではなくブラウザの仕組みで解決する
3. **判断基準の明示** — 各層に「何を書くか」の明確な判断基準がある
4. **CSS の進化への追従** — 層構成は CSS の進化に応じて適応させる設計余地を持つ（具体的な検討事項は README を参照）

---

## 2. Conformance

*This section is normative.*

本文書で *This section is normative.* と記されたセクションは準拠義務のある規定、*This section is informative.* と記されたセクションは参考情報である。

### 適用範囲

*This section is normative.*

本仕様の要求レベル（MUST / MUST NOT / SHOULD / SHOULD NOT / MAY）および §6 の命名規則は、**mFLOCSS が管理するクラス**にのみ適用される。mFLOCSS が管理するクラスとは、§6 の命名規則に従うクラス（`l-*` / `c-*` / `p-*` / `u-*` / `a-*`）、および `@layer` の各層定義として記述されるクラスを指す。加えて、§7 の HTML マークアップに関する規定（MUST NOT [静的インラインスタイルの禁止] 等）はプロジェクト全体に適用される。

CMS・フレームワーク・外部ライブラリ・外部サービス等が生成するクラスで、mFLOCSS 命名規則に従わないものを **外部生成クラス** と呼ぶ。外部生成クラスへのクラスセレクタ使用、および外部生成クラスとの組み合わせは、本仕様の要求レベル・命名規則の制約を受けない。

### 要求レベル

本仕様のキーワード MUST / MUST NOT / SHOULD / SHOULD NOT / MAY は RFC 2119 [RFC2119]（RFC 8174 [RFC8174] により補足）に基づく。

| キーワード | 意味 |
|---|---|
| MUST | 絶対的な要求。違反は非準拠となる |
| MUST NOT | 絶対的な禁止。違反は非準拠となる |
| SHOULD | 推奨。正当な理由がある場合に限り逸脱を許容する |
| SHOULD NOT | 非推奨。正当な理由がある場合に限り逸脱を許容する |
| MAY | 任意。プロジェクトの判断に委ねる |

> **注記（Informative）**: SHOULD からの逸脱は、不変原則（§1.2）を理解した上で判断すること。逸脱の理由をコード内コメントまたは設計文書に残すことを推奨する。

### 準拠条件

mFLOCSS に準拠するとは、本仕様の全 MUST / MUST NOT ルールに違反しないことを意味する。

MUST は `@layer` の層構造カスケードを保護するルール、層間の依存方向を保護するルール、または Utility 層の最終上書きを構造的に保証するルール（他ルール違反時のフェイルセーフ）に使用する。

設計判断の品質（層の選択・Portability Test 等）は SHOULD で推奨する。

> **注記（Informative）**: SHOULD ルールを遵守するほど設計の一貫性と保守性が向上する。

### バージョニング

- メジャー: 層の追加・削除・統合。MUST/MUST NOT の追加・削除・変更（他レベルからの昇格・降格を含む）
- マイナー: SHOULD/MAY の追加・削除・レベル間の変更。Informative セクションの変更
- パッチ: 誤記修正・Example の改善等、要求レベルに影響しない変更

---

## 3. Layer Architecture

*This section is normative.*

mFLOCSS は以下の層で構成される。本章以降で使用する Block・Element 等の命名規則は §6 で定義する。

| 順序 | 層名 | プレフィックス | 責任 |
|---|---|---|---|
| 1 | token | — | デザイントークンの定義（プリミティブ変数・セマンティック変数・グローバルトークン）および計算ヘルパーの管理 |
| 2 | reset | — | ブラウザデフォルトの初期化 |
| 3 | foundation | — | 要素の基本スタイル |
| 4 | layout | `l-` | 位置と空間の配置 |
| 5 | component | `c-` | 配置先に左右されない再利用可能なパーツ |
| 6 | project | `p-` | サイト固有のパーツとデザイン要件 |
| 7 | animation | — | 動きの分離管理 |
| 8 | utility | `u-` | 単一目的のスタイル上書き |

### 層間の依存方向

テーブルの下にある層ほど @layer の優先度が高い。上位層とは §3 層テーブルの順序番号が大きい層（`@layer` 優先度が高い）、下位層とは順序番号が小さい層を指す。例: Utility（8）が最上位、Token（1）が最下位。

**要求レベル:**

- MUST NOT [逆方向参照の禁止]: CSS セレクタおよびカスタムプロパティの参照において、下位層から上位層のクラスやカスタムプロパティを参照してはならない（例: Component 層が Project 層のクラスに依存してはならない）

> **注記（Informative）**: `@layer` の先制宣言では後に宣言した層ほど優先されるため、テーブルの下にある層ほど優先度が高い。Utility を最後に宣言することで最高優先度が保証される。

> **Example（MUST NOT [逆方向参照の禁止]）:**

```html
<div class="c-modal">
  <form class="p-login-form">
    <button class="c-button">送信</button>
  </form>
</div>
```

> **注記（Informative）**: HTML の入れ子構造は本ルールの対象外である。上記の例は CSS で他層のクラスを参照していないため違反にならない。Component は Component のままで任意の層の要素を内包でき、Project も同様である。

### Layer Judgment Flowchart

> **注記（Informative）**: このフローチャートは層判断の参考ガイドであり、規範的ルールではない。

スタイルをどの層に書くべきかを 7 ステップで判断する。

```
Step 1: デザイントークン（色の値、フォント名、z-index 値等）または計算ヘルパー（--px 等）か？
  └─ Yes → Token（プリミティブ変数とセマンティック変数を同層で管理）

Step 2: ブラウザデフォルトの初期化（リセット CSS）か？
  └─ Yes → Reset

Step 3: 全ページ共通の基本スタイルか？
  └─ Yes → Foundation

Step 4: 配置と空間だけか？（色・文字・装飾に触れないか？）
  └─ Yes → Layout

Step 5: Portability Test — 別のサイトにそのまま持っていけるか？
  ├─ Yes → Component
  └─ No → Project
  ※ 判断が曖昧な場合は、§5.5 の補助テスト（Responsibility Test）を使用する。

※ Step 6-7 は Step 1-5 の判定を覆さない追加チェックである。
  Step 1-5 で判定した層のスタイルの中に、Animation・Utility の条件を
  満たす部分があれば、その部分を該当層に分離する。
  （例: Step 5 で Component と判定したスタイルに装飾的アニメーションが
  含まれる場合、その動きの部分だけを Animation 層に切り出す）

Step 6: 装飾的アニメーション（視覚演出）か？
  ├─ Yes → Animation（2 ガード原則を適用）
  └─ 機能的トランジション（インタラクションフィードバック）→ Component / Project に残す

Step 7: 特定の Block に帰属しない、単一目的の微調整か？
  └─ Yes → Utility
```

---

## 4. Layer Order Declaration

*This section is normative.*

CSS `@layer` の構造的整合性を維持するために必要な宣言ルールを定義する。先制宣言・`!important` の使用制限・外部 CSS の取り込みを規定する。

**要求レベル:**

- MUST [先制宣言の実施]: CSS Cascading and Inheritance Level 5 [CSS-CASCADE-5] に定義される `@layer` による層間の優先順位宣言を起点ファイル（エントリポイント）CSS の先頭で、全ての `@import` に先行して行わなければならない。
- MUST [外部 CSS の層取り込み]: 外部 CSS は `@import url() layer()` を使用するか、バンドラーを使用する場合は `@layer` 宣言内にバンドル結果が配置されるように構成し、いずれかの層に取り込まなければならない。
- MUST NOT [!important の使用制限]: Reset 層および Utility 層を除く全層で `!important` を使用してはならない。Reset 層の内部実装における `!important` の有無は本仕様の準拠対象外とする。

> **注記（Informative）**: Reset 層は外部リセット CSS を取り込むためスコープ除外とする。この制約により優先度逆転の複雑性を回避する。

> **Example（MUST [先制宣言の実施]）:**

```css
@layer token, reset, foundation, layout, component, project, animation, utility;
```

> **Example（MUST [外部 CSS の層取り込み]）:**

```css
@import url("vendor/reset.css") layer(reset);
```

---

## 5. Layer Definitions

*This section is normative.*

### 5.1 Token

**Token 層の責任:**

1. **デザイントークンの定義**: プリミティブ変数（生の値）・セマンティック変数（意味を持つマッピング）・グローバルトークンの管理
2. **ブランドトークンとグローバルトークンの分離**: プロジェクト固有の値と汎用的な値を区別する
3. **計算ヘルパーの管理**: 単位変換・計算のためのユーティリティ変数（`--px` 等）の管理

**検証問い（Token Test）:**

「この値はデザイントークン（色の値、フォント名、余白、z-index 等）か？または計算ヘルパー（単位変換・計算のためのユーティリティ変数。`--px` 等）か？」

- Yes → Token
- No → 他の層

**要求レベル:**

- SHOULD [Token 層の責任限定]: Token 層を使用する場合、デザイントークン（プリミティブ値・セマンティック変数・グローバルトークン）および計算ヘルパーの定義に限定すべきである
- SHOULD [:root セレクタ限定]: `:root` セレクタのみを使用すべきである
- MAY [テーマ切替の Token 完結]: ダークモード / テーマ切替は Token 層のセマンティック変数で完結させてよい

**トークンの分類:**

> **注記（Informative）**: 以下はトークンの分類の参考整理である。Token 層はプリミティブ変数とセマンティック変数を同一層内で管理し、コンテキスト依存でないカテゴリは値のみを定義する。分類ごとの具体的な参照ルールは §7 を参照。

| 分類 | 性質 | 例 |
|------|------|-----|
| プリミティブ変数 | 生の値の定義 | `--_slate-600`, `--_slate-900` |
| セマンティック変数 | 意味を持つマッピング | `--color-main`, `--color-surface`, `--font-size-body` |
| グローバルトークン | 多くのプロジェクトで共通して使える値 | `--ease-out-cubic`, `--z-header` |
| 計算ヘルパー | 単位変換・計算のためのユーティリティ変数 | `--px: calc(1rem / 16)`（`calc(数値 * var(--px))` で rem に変換） |

命名規則は §6 カスタムプロパティ命名まとめを参照。

> **Example（SHOULD [:root セレクタ限定]）:**

```css
@layer token {
  :root {
    /* プリミティブ（color） */
    --_slate-900: #1a202c;
    --_slate-100: #f1f5f9;

    /* セマンティック */
    color-scheme: light dark;
    --color-main: light-dark(var(--_slate-900), var(--_slate-100));
    --font-family: "Noto Sans JP", sans-serif;

    /* グローバルトークン */
    --ease-out-cubic: cubic-bezier(0.33, 1, 0.68, 1);
  }
}
```

### 5.2 Reset

**Reset 層の責任:**

1. **ブラウザデフォルトの初期化**: リセットまたはノーマライズによるブラウザ間差異の吸収

**検証問い（Reset Test）:**

「このスタイルはブラウザデフォルトの初期化か？」

- Yes → Reset
- No → Foundation 以降の層

**要求レベル:**

- SHOULD [Reset 層の責任限定]: Reset 層を使用する場合、ブラウザデフォルトの初期化に限定すべきである（自作・外部を問わない）
- SHOULD NOT [プロジェクト固有スタイルの Reset 記述禁止]: プロジェクト固有のスタイルを Reset 層に記述すべきでない
- MAY [Reset 層の使用任意]: Reset 層の使用は任意である

> **Example（SHOULD [Reset 層の責任限定]）:**

```css
@layer reset {
  *,
  ::before,
  ::after {
    box-sizing: border-box;
  }

  :where(body) {
    margin: unset;
  }

  :where(ul, ol, menu) {
    padding-inline-start: unset;
    list-style-type: '';
  }
}
```

### 5.3 Foundation

**Foundation 層の責任:**

1. **全ページ共通の基本スタイル定義**: 要素型セレクタによる基本スタイル（body のフォント、見出しの基本サイズ、リンク色等）の付与と、テーマが管理しない出力（CMS・フレームワーク等の外部生成クラス）の正規化

**検証問い（Foundation Test）:**

「このスタイルは、全ページ共通の基本スタイルか？」

- Yes → Foundation
- No → 上位層

**要求レベル:**

- SHOULD [Foundation 層の責任限定]: Foundation 層を使用する場合、全ページ共通の基本スタイルの定義（要素型セレクタによる基本スタイルと外部生成クラスの正規化）に限定すべきである
- SHOULD NOT [Component/Project スタイルの Foundation 記述禁止]: Component 層または Project 層に属するスタイルを Foundation 層に記述すべきでない

> **注記（Informative）**: 特定のコンテキストに依存するスタイルは Foundation ではなく上位層に記述する。

> **Example（SHOULD NOT [Component/Project スタイルの Foundation 記述禁止]）:**

```css
@layer foundation {
  :where(body) {
    font-family: var(--font-family);
    color: var(--color-main);
    background-color: var(--color-surface);
    line-height: 1.5;
  }

  :where(input, select, textarea, button) {
    font: inherit;
  }
}
```

### 5.4 Layout

**Layout 層の責任:**

1. **ページの骨格（ストラクチャ）**: ヘッダー、メインコンテンツ、セクション、フッターなど、ページ全体の構造を定義する
2. **空間の確保**: セクション間の余白（`padding-block`）、コンテンツ幅の制約（`max-inline-size`）
3. **配置制御**: `position: sticky` / `z-index` などの構造的な配置

**プレフィックス:** `l-`

**検証問い（Layout Test）:**

「このスタイルは、配置・寸法・空間の確保か？ 中身の見た目は変わらないか？」

- Yes → Layout
- No → 他層

**要求レベル:**

- SHOULD [Layout 層の責任限定]: Layout 層を使用する場合、配置・寸法・空間の確保に限定すべきである
- SHOULD NOT [視覚プロパティの排除]: 見た目に関するプロパティ（`color`, `font-size`, `background-color`, `border`, `text-align` 等の視覚プロパティ）を宣言すべきでない

> **注記（Informative）**: 中身の見た目（色・文字・装飾・影・透明度）が変わる場合は Layout ではない。

> **Example（SHOULD [Layout 層の責任限定]）:**

```css
@layer layout {
  /* 配置・重なり */
  .l-header {
    position: sticky;
    inset-block-start: 0;
    z-index: var(--z-header);
  }

  /* 公開 API: カスタムプロパティで上位層に値の設定を委ねる */
  .l-section {
    --section-padding: 3.75rem;

    padding-block: var(--section-padding);
  }
}
```

> **注記（Informative）**: Container Queries の `container-type` / `container-name` 宣言は、対象となる Block を記述する層で行う。Layout 層に限定されない。

### 5.5 Component

**Component 層の責任:**

1. **再利用可能なパーツの定義**: 配置先に依存しない、自己完結した UI パーツ

**プレフィックス:** `c-`

**検証問い（Portability Test）:**

「そのパーツを別のサイトにそのまま持っていけるか？」

- Yes → Component
- No → Project（§5.6）

**補助テスト（Responsibility Test）:**

Portability Test で判断が曖昧な場合にのみ使用する。Portability Test が明確に Yes または No を返す場合、Responsibility Test の結果に関わらずその判定に従う。

> 「このスタイルは、パーツ自身の視覚的責任か？」

- Yes → Component: ボタンの背景色、カードの角丸
- No（使う側のデザイン要件）→ Project: ヒーロー内のボタンのサイズ変更、カードの配置間隔

**要求レベル:**

- SHOULD [Portability Test の合格]: Portability Test に合格すべきである
- SHOULD NOT [外部レイアウト影響プロパティの排除]: Component のルート要素（= Component を構成する HTML の最外殻要素）に、外部レイアウトに影響するプロパティ（`margin`, `position: absolute/fixed/sticky`）を含めるべきでない
- SHOULD NOT [存在/不在制御の排除]: Component のルート要素に、自身の存在/不在を制御するプロパティ（`display: none`）を含めるべきでない

> **注記（Informative）**: 上記 2 つの SHOULD NOT はいずれも Portability Test 不合格の代表例である。配置や存在の制御は「それは Component 自身の責任か、使う側の責任か？」（Responsibility Test）の観点で使う側の責任に該当する。存在/不在の制御は HTML 属性（`hidden`、`<dialog>` の `showModal()`/`close()`）または Project 層で行う。Utility 層の `u-visually-hidden` 等の単一目的スタイルは §5.8 Utility 責任に従う。

> **Example（SHOULD [Portability Test の合格]）:**

```css
@layer component {
  .c-card {
    /* セマンティック変数を参照 */
    padding: var(--space-lg);
    background-color: var(--color-surface);

    /* プライベートカスタムプロパティ（§7）: Block 内部の計算用 */
    --_gap: var(--space-md);
    display: grid;
    gap: var(--_gap);
  }

  /* Element */
  .c-card__title {
    font-size: var(--font-size-lg);
  }

  /* Modifier: 静的なバリエーション */
  .c-card.-compact {
    --_gap: var(--space-sm);
    padding: var(--space-sm);
  }
}
```

### 5.6 Project

**Project 層の責任:**

1. **サイト固有のパーツとデザイン要件**: Portability Test で No と判断されるパーツのスタイル（別のサイトにそのまま持っていけない、そのサイト固有のデザイン）

**プレフィックス:** `p-`

**検証問い:**

Portability Test（§5.5）で No → Project

**要求レベル:**

- SHOULD [Project Block ルートの付与]: サイト固有のデザイン要件があるパーツ（セクション、サイト共通のヘッダー / フッター、ドロワー、サイト固有のナビゲーション等）のルート要素に Project Block を付与し、構成要素は Element として構築すべきである
- SHOULD NOT [Component 相当スタイルの Project 記述禁止]: Portability Test（§5.5）に合格するスタイルを Project 層に記述すべきでない。該当するスタイルは Component 層（§5.5）に記述する

> **注記（Informative）**: Project は独自の Test を持たず、Portability Test の否定形で定義される唯一の層である。

> **Example（SHOULD [Project Block ルートの付与]）:**

```css
@layer project {
  /* セクションルートへの Project Block */
  .p-about {
    --section-padding: 2.5rem;
  }

  /* Element: Layout の配置先適応 */
  .p-about__inner {
    max-inline-size: 50rem;
  }

  /* サイト共通パーツへの Project Block（セクション以外の例） */
  .p-site-header {
    position: sticky;
    inset-block-start: 0;
  }

  .p-site-header__logo {
    block-size: 2rem;
  }

  /* Project 固有のパーツ（セクション内要素） */
  .p-hero__lead {
    text-align: center;
  }
}
```

### 5.7 Animation

**Animation 層の責任:**

1. **装飾的アニメーションの分離管理**: Component / Project から動きを分離し一元管理
2. **状態遷移の制御**: `data-*` 属性による表示制御

**検証問い（Animation Test）:**

「その動きを無効化しても、インタラクションの意味が伝わるか？」

- Yes（なくても伝わる）→ 装飾的 → Animation
- No（ないと操作感が損なわれる）→ 機能的 → Component/Project

**要求レベル:**

- SHOULD [装飾的アニメーションの Animation 層分離]: 装飾的アニメーションは Animation 層に分離し、2 ガード原則を適用すべきである
- SHOULD [2 ガード原則の実装]: Animation 層のスタイルは、`prefers-reduced-motion: reduce` 環境でアニメーションが無効になり、`scripting: none` 環境で要素が不可視にならない実装とすべきである
- SHOULD [translate / rotate / scale を含む機能的トランジションのガード]: 機能的トランジションのうち `translate` / `rotate` / `scale` を含むものは、`prefers-reduced-motion: no-preference` のガードを適用すべきである。色変化（`color` / `background-color` 等）や `opacity` のみのトランジションはガード不要である
- SHOULD [@keyframes 名と data-* 属性値の一致]: `@keyframes` 名は対応する `data-*` 属性の値と一致させるべきである（§5.7 @keyframes 命名を参照）
- MAY [機能的トランジションの所属層への記述]: 機能的トランジションは、対象の Block が属する層（Component または Project）に記述してよい

> **注記（Informative）**: `scripting` は Media Queries Level 5 [MEDIAQUERIES-5] で定義されるメディア特性であり、JS の有効・無効を CSS で検出する。未対応ブラウザでは統合ガード全体が false に評価されアニメーション CSS が一切適用されない（要素は可視状態を維持するため安全性は確保される）。

> **注記（Informative）**: `translate` / `rotate` / `scale` を含むトランジションは前庭障害のトリガーになりうるため、ガード適用を推奨する。

#### セレクタ

Animation 層は `data-*` 属性セレクタを使用する。

> **注記（Informative）**: 属性名はプロジェクトで任意に決められる。本仕様の例では慣例として `data-animate` を用いる。

#### 動きの分類

Animation 層に分離すべき動きと、Component/Project に残してよい動きを区別する。

| 分類 | 性質 | 例 | 層 |
|---|---|---|---|
| 装飾的アニメーション | 視覚演出。なくても機能に影響しない | fade-in, slide-up, parallax | Animation |
| 機能的トランジション | インタラクションフィードバック。ユーザー操作に対する応答 | hover の色変化, ボタンの translate, フォーカスリングの遷移 | Component / Project |

#### @keyframes 命名

`@layer` は `@keyframes` 名のスコープを分離しないため、属性値との対応関係を明確にすることで名前衝突のリスクを低減する（例: `data-animate="scale-in"` → `@keyframes scale-in`）。

> **Example（SHOULD [2 ガード原則の実装]、SHOULD [@keyframes 名と data-* 属性値の一致]）:**

```css
@layer animation {
  /* 統合ガード: 2 ガード原則を一括適用 */
  @media (prefers-reduced-motion: no-preference) and (scripting: enabled) {
    @keyframes fade-in {
      from { opacity: 0; }
    }
    [data-animate="fade-in"] {
      animation: fade-in 0.6s var(--ease-out-cubic) both;
      animation-play-state: paused;
    }
    /* JS（IntersectionObserver 等）が data-visible を付与してアニメーション開始 */
    [data-animate="fade-in"][data-visible] {
      animation-play-state: running;
    }
  }
}
```

### 5.8 Utility

**Utility 層の責任:**

1. **単一目的のスタイル上書き**: 1 プロパティ（または密接に関連する最小セット）で完結
2. **Block に帰属しないグローバルな補助**: 特定の Component に属さない汎用的なスタイル
3. **最終上書きの保証**: `!important` による全層に対する優先

**プレフィックス:** `u-`

**検証問い（Utility Test）:**

「このスタイルは特定の Block に帰属せず、単一プロパティ（または密接に関連する最小プロパティセット）で完結するか？」

- Yes → Utility
- No → Component / Project

**要求レベル:**

- MUST [!important の付与]: Utility クラスの各宣言プロパティに `!important` を付与しなければならない（使用制限については §4 を参照）
- SHOULD [Utility 層の責任限定]: Utility 層を使用する場合、特定の Block に帰属しない横断的かつ局所的な単一目的のスタイルに限定すべきである
- SHOULD NOT [Block 帰属スタイルの Utility 記述禁止]: 特定の Block や Element に帰属できるスタイルを Utility に書くべきでない — Utility は特定の Block に帰属しない横断的かつ局所的な単一目的のスタイルに限る
- MAY [セマンティックなファイルグループ化]: セマンティックな意味でファイルをグループ化してよい（例: `u-hidden.css` に `u-visually-hidden` と `u-hidden-sp` をまとめる）

> **Example（MUST [!important の付与]、SHOULD NOT [Block 帰属スタイルの Utility 記述禁止]）:**

```css
@layer utility {
  .u-visually-hidden:not(:focus, :active, :focus-within) {
    position: absolute !important;
    inline-size: 1px !important;
    block-size: 1px !important;
    padding: 0 !important;
    margin: -1px !important;
    overflow: hidden !important;
    white-space: nowrap !important;
    border: 0 !important;
    clip-path: inset(50%) !important;
  }

}
```

---

## 6. Naming Conventions

*This section is normative.*

mFLOCSS は [FLOCSS] が採用する BEM 命名規則をベースとし、以下の mFLOCSS 固有の変更を定義する。Block は独立した再利用可能な単位、Element は Block に従属する構成要素であり `.{prefix}-{block}__{element}` 形式で表現する。Modifier は BEM の `--` ではなく `.-modifier` 形式を採用する。

**要求レベル:**

- SHOULD [層識別プレフィックスの使用]: 層の識別のためにプレフィックスを使用すべきである。Token・Reset・Foundation・Animation はプレフィックスの対象外とする
- SHOULD NOT [Element 連鎖の回避]: Element 名を連鎖させるべきではない（例: `block__elem1__elem2`）

> **注記（Informative）**: Token・Reset・Foundation はクラスセレクタを使用しないため、Animation は `data-*` 属性セレクタを使用するため（§5.7 セレクタを参照）、プレフィックスの対象外としている。`@scope` [CSS-CASCADE-6] 等の機能によりプレフィックスが不要になる可能性があるため、本項は MUST としない。

### クラス名

```
.{prefix}-{block}__{element}
.{prefix}-{block}.-{modifier}
.{prefix}-{block}__{element}.-{modifier}
```

| 要素 | 形式 | 例 |
|---|---|---|
| Block | `.{prefix}-{block}` | `.c-button`, `.l-section` |
| Element | `.{prefix}-{block}__{element}` | `.c-card__title` |
| Modifier | `.-{modifier}` | `.c-button.-primary`, `.l-section.-wide` |
| State（data-*） | `[data-{state}]` | `[data-active]`, `[data-loading]`, `[data-visible]` |
| State（ARIA） | `[aria-{prop}="..."]` | `[aria-expanded="true"]`, `[aria-current="page"]` |

### 静的バリエーション（Modifier）

`.-modifier` 形式。Block または Element と併用する。HTML に記述し、原則として変化しない。

### 動的状態（State）

`data-*` 属性または ARIA 属性（`aria-expanded`, `aria-current` 等）で表現する。JS やユーザー操作により変化する。Block・Element・Animation の `data-*` 属性いずれとも組み合わせて使用できる（例: `.c-button[data-loading]`、`[data-animate="fade-in"][data-visible]`）。

### カスタムプロパティ命名まとめ

| 層 | パターン | 例 |
|---|---|---|
| Token（プリミティブ） | `--_{カテゴリ}-{名前}` | `--_slate-600` |
| Token（セマンティック） | `--{カテゴリ}-{役割}` | `--color-main`, `--font-size-body` |
| Token（グローバルトークン） | `--{カテゴリ}-{名前}` | `--ease-out-cubic`, `--z-header` |
| Token（計算ヘルパー） | `--{名前}` | `--px` |
| 公開 API | `--{対象}-{名前}` | `--section-padding`, `--badge-bg` |
| プライベートカスタムプロパティ | `--_{名前}` | `--_font-size-min` |

---

## 7. Custom Properties and Inline Styles

*This section is normative.*

カスタムプロパティのトークン参照チェーン（Token 層内のプリミティブ → セマンティックの参照パスと、Foundation 以降の層によるセマンティック変数の参照ルール）、およびインラインスタイルの使用制限を定義する。

**要求レベル:**

- MUST NOT [静的インラインスタイルの禁止]: HTML マークアップに静的なインラインスタイルを記述してはならない。JS による動的なスタイル注入や、CMS・ライブラリが自動生成するインラインスタイルは本規定の対象外とする

> **注記（Informative）**: インラインスタイルはどの `@layer` にも属さない unlayered CSS として扱われ、全ての layered CSS より優先されるため、層構造による優先順位制御を破壊する。
- SHOULD [セマンティック変数経由の参照]: Foundation 以降の層はブランドトークンについて Token 層のセマンティック変数を参照すべきである
- SHOULD [公開 API 変数の命名規則遵守]: 上位層（Project 等）から値を設定する変数、または JS から値を注入する変数は、`--{対象}-{名前}` の公開 API 命名を使用すべきである
- SHOULD NOT [プリミティブ変数の直接参照禁止]: Token 層以外がプリミティブ変数（Token 層の `--_` プレフィックス変数）を直接参照すべきでない
- MAY [グローバルトークンの直接参照]: グローバルトークンおよび計算ヘルパー（`--px` 等）は Foundation 以降の層から直接参照してよい（§5.1 トークン分類を参照）
- MAY [プライベートカスタムプロパティの定義]: Layout 以降の層でプライベートカスタムプロパティ（`--_xxx`）を定義してよい

> **Example（SHOULD [セマンティック変数経由の参照]、SHOULD [公開 API 変数の命名規則遵守]）:**

```css
/* Token: セマンティック変数を定義 */
@layer token {
  :root {
    --space-md: 1rem;
    --space-lg: 1.5rem;
  }
}

/* Component: Token を参照し、公開 API を定義 */
@layer component {
  .c-card {
    --card-padding: var(--space-md);
    padding: var(--card-padding);
  }
}

/* Project: Component の公開 API を上書き */
@layer project {
  .p-about {
    --card-padding: var(--space-lg);
  }
}
```

---

## 8. File Architecture

*This section is normative.*

ファイルとディレクトリの構造・命名・エントリポイントの構成を定義する。物理的なファイル配置と層構造の対応を規定する。

**要求レベル:**

- SHOULD [ディレクトリ名の層名一致]: ディレクトリ名は層名と一致させるべきである
- SHOULD [1 Block = 1 ファイルの維持]: 1 つの CSS ファイルには 1 つの Block を定義すべきである
- SHOULD [層帰属の明確化]: 各ファイルがどの層に属するかを明確にすべきである。方法はプロジェクトの規模やツールチェーンに応じて選択してよい

> **注記（Informative）**: 複数の独立した Block を 1 ファイルに含めると、ファイル名と Block の対応が崩れる。

> **注記（Informative）**: 以下のディレクトリ構造・ファイル例は推奨される一例である。プロジェクトの要件に応じて構成を変更してよい。

### ディレクトリ構造

> **Example（SHOULD [ディレクトリ名の層名一致]）:**

```
css/
├── layer-order.css       /* 層の先制宣言（§4） */
├── property.css          /* @property 宣言（任意） */
├── style.css             /* エントリポイント */
├── token/                /* §5.1 Token — デザイントークン */
├── reset/                /* §5.2 Reset — ブラウザデフォルト初期化 */
├── foundation/           /* §5.3 Foundation — 全ページ共通の基本スタイル */
├── layout/               /* §5.4 Layout — ページの骨格・配置 */
├── component/            /* §5.5 Component — 再利用可能なパーツ */
├── project/              /* §5.6 Project — サイト固有のスタイル */
├── animation/            /* §5.7 Animation — 装飾的アニメーション */
└── utility/              /* §5.8 Utility — 単一目的の上書き */
```

### 1 Block = 1 ファイル

> **注記（Informative）**: Utility 層は例外とする。Utility は Block 構造を持たない単一目的のクラスであり、セマンティックな意味でグループ化することが推奨される（§5.8 参照）。

### layer-order.css

> **注記（Informative）**: `@layer` の先制宣言のみを含む。スタイル定義は置かない。層ディレクトリ（`token/` 等）と同列に配置する。

### property.css

> **注記（Informative）**: `@property` 宣言（[CSS-PROPERTIES-VALUES-1]）を含む。`@property` の登録スコープに関する W3C 仕様が未確定であり、ビルドツールによる非対応も報告されているため、`@layer` の外に配置する。層ディレクトリには入れない。`@property` を使用しないプロジェクトではこのファイルは不要である。

---

## Appendix A: Glossary

*This section is normative.*

| 用語 | 定義 |
|---|---|
| **外部生成クラス** | CMS・フレームワーク・外部ライブラリ・外部サービス等が生成するクラスで、mFLOCSS 命名規則に従わないもの。本仕様の要求レベル・命名規則の制約を受けない（初出: §2） |
| **上位層** | §3 層テーブルの順序番号が大きい層。@layer 優先度が高い。例: Utility（8）が最上位（初出: §3） |
| **下位層** | §3 層テーブルの順序番号が小さい層。@layer 優先度が低い。例: Token（1）が最下位（初出: §3） |
| **デザイントークン** | Token 層で管理するすべての変数の総称。プリミティブ変数・セマンティック変数・グローバルトークンを含む（計算ヘルパーは含まない）（初出: §3） |
| **先制宣言** | `layer-order.css` における `@layer` による層間の優先順位宣言。全スタイル定義に先行して記述される（初出: §4） |
| **Token Test** | Token 層の適用可否を判定する検証問い（§5.1） |
| **ブランドトークン** | プロジェクトごとに変わるデザイン値（color, typography, structure）。プリミティブ変数とセマンティック変数の総称。参照ルールは §5.1 および §7 を参照（初出: §5.1） |
| **プリミティブ変数** | `--_` プレフィックスを持つ Token 層の変数。生の値（HEX 色値、px 数値等）を保持する。ブランドトークンの一種（初出: §5.1） |
| **セマンティック変数** | 意味を持つ Token 層のマッピング変数（`--color-main` 等）。プリミティブ変数を参照でき、コンテキスト（ダークモード等）に応じた役割を表現する。ブランドトークンの一種（初出: §5.1） |
| **グローバルトークン** | 多くのプロジェクトで共通して使える値（ease, z-index, font-weight）。計算ヘルパーは独立したカテゴリであり、グローバルトークンには含まない。参照ルールは §5.1 および §7 を参照（初出: §5.1） |
| **計算ヘルパー** | 単位変換・計算のためのユーティリティ変数。Token 層に配置する（初出: §5.1） |
| **Reset Test** | Reset 層の適用可否を判定する検証問い（§5.2） |
| **Foundation Test** | Foundation 層の適用可否を判定する検証問い（§5.3） |
| **Layout Test** | Layout 層の適用可否を判定する検証問い（§5.4） |
| **Portability Test** | Component と Project の境界を判定する検証問い（§5.5） |
| **Responsibility Test** | Portability Test の補助テスト。判断が曖昧な場合にのみ使用（§5.5） |
| **外部レイアウト影響プロパティ** | Component のルート要素に指定した場合、配置先レイアウトに影響するプロパティ（`margin`, `position: absolute/fixed/sticky` 等）（§5.5） |
| **セクションルート** | ページ内の各セクションを包む最外殻要素（初出: §5.6） |
| **Animation Test** | Animation 層の適用可否を判定する検証問い（§5.7） |
| **装飾的アニメーション** | 視覚演出としての動き。無効化しても機能に影響しない。Animation 層に分離し、2 ガード原則を適用する（初出: §5.7） |
| **機能的トランジション** | インタラクションフィードバックとしての動き。ユーザー操作に対する応答であり、対象の Block が属する層（Component または Project）に記述する。`translate` / `rotate` / `scale` を含む場合は `prefers-reduced-motion` ガードを適用する（初出: §5.7） |
| **2 ガード原則** | Animation 層で `prefers-reduced-motion` と `scripting` の 2 条件を考慮すること。推奨は `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガード（初出: §5.7） |
| **統合ガード** | 2 ガード原則の推奨実装パターン。`@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` で Animation 層のスタイル全体をラップし、条件を満たさない場合にブロック全体を不適用にする方式（初出: §5.7） |
| **Utility Test** | Utility 層の適用可否を判定する検証問い（§5.8） |
| **Block** | BEM における独立した意味のあるエンティティ。プレフィックス付きクラス名（`.c-card`, `.p-hero` 等）で表現する（初出: §6） |
| **Element（`__element`）** | Block の一部。命名は `.{prefix}-{block}__{element}` の形式。Block なしでの使用回避等の規範的定義は §6 を参照（初出: §6） |
| **Modifier（`.-xxx`）** | 静的なバリエーション。定義は §6 を参照（初出: §6） |
| **State** | data-* 属性または ARIA 属性で表現する動的な状態。JS やユーザー操作により変化する（初出: §6） |
| **トークン参照チェーン** | ブランドトークンの参照パス。Token 層内（プリミティブ → セマンティック）→ Foundation 以降の層（セマンティック変数を参照）。グローバルトークンは Foundation 以降から直接参照してよい（§7 MAY）。参照ルールは §7 で規定する（初出: §7） |
| **unlayered CSS** | `@layer` ブロック外に記述されたスタイル。全ての layered CSS より優先される（初出: §7） |
| **公開 API（カスタムプロパティ）** | `--{対象}-{名前}` 形式の変数。上位層または JS から上書きされることを想定する外部インターフェース（初出: §7） |
| **プライベートカスタムプロパティ** | `--_` プレフィックスを持つ変数。Block または Element 内部でのみ使用し、外部からの参照・設定を想定しない（初出: §7）。「プライベート変数」は本用語の略称 |

---

## Appendix B: References

### Normative References

*This section is normative.*

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997. https://www.rfc-editor.org/rfc/rfc2119
- **[RFC8174]** Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words", BCP 14, RFC 8174, May 2017. https://www.rfc-editor.org/rfc/rfc8174
- **[CSS-CASCADE-5]** Atkins Jr., T.; Rivoal, F.; Lilley, C., "CSS Cascading and Inheritance Level 5", W3C Candidate Recommendation.
- **[CSS-PROPERTIES-VALUES-1]** Atkins-Bittner, T.; Stearns, A.; Whitworth, G., "CSS Properties and Values API Level 1", W3C Working Draft.
- **[MEDIAQUERIES-5]** Rivoal, F.; Khan, A. A., "Media Queries Level 5", W3C Working Draft. https://www.w3.org/TR/mediaqueries-5/

### Informative References

*This section is informative.*

- **[FLOCSS]** Hiloki, "FLOCSS — Foundation Layout Object CSS". https://github.com/hiloki/flocss
- **[CSS-CASCADE-6]** Atkins Jr., T.; Rivoal, F.; Lilley, C., "CSS Cascading and Inheritance Level 6", W3C Working Draft. https://www.w3.org/TR/css-cascade-6/

---

## Appendix C: Changes

*This section is informative.*

変更履歴は [CHANGELOG.md](./CHANGELOG.md) を参照。

---

## Appendix D: Requirements Index

*This section is informative.*

本仕様に含まれる全要求事項の一覧。各項目は本文の該当箇所への参照を含む。

### MUST / MUST NOT

| § | 一言サマリ | 全文参照 |
|---|---|---|
| §3 | MUST NOT [逆方向参照の禁止] | §3 層間の依存方向 |
| §4 | MUST [先制宣言の実施] | §4 要求レベル |
| §4 | MUST [外部 CSS の層取り込み] | §4 要求レベル |
| §4 | MUST NOT [!important の使用制限] | §4 要求レベル |
| §5.8 | MUST [!important の付与] | §5.8 要求レベル |
| §7 | MUST NOT [静的インラインスタイルの禁止] | §7 要求レベル |

### SHOULD / SHOULD NOT

| § | 一言サマリ | 全文参照 |
|---|---|---|
| §5.1 | SHOULD [Token 層の責任限定] | §5.1 要求レベル |
| §5.1 | SHOULD [:root セレクタ限定] | §5.1 要求レベル |
| §5.2 | SHOULD [Reset 層の責任限定] | §5.2 要求レベル |
| §5.2 | SHOULD NOT [プロジェクト固有スタイルの Reset 記述禁止] | §5.2 要求レベル |
| §5.3 | SHOULD [Foundation 層の責任限定] | §5.3 要求レベル |
| §5.3 | SHOULD NOT [Component/Project スタイルの Foundation 記述禁止] | §5.3 要求レベル |
| §5.4 | SHOULD [Layout 層の責任限定] | §5.4 要求レベル |
| §5.4 | SHOULD NOT [視覚プロパティの排除] | §5.4 要求レベル |
| §5.5 | SHOULD [Portability Test の合格] | §5.5 要求レベル |
| §5.5 | SHOULD NOT [外部レイアウト影響プロパティの排除] | §5.5 要求レベル |
| §5.5 | SHOULD NOT [存在/不在制御の排除] | §5.5 要求レベル |
| §5.6 | SHOULD [Project Block ルートの付与] | §5.6 要求レベル |
| §5.6 | SHOULD NOT [Component 相当スタイルの Project 記述禁止] | §5.6 要求レベル |
| §5.7 | SHOULD [装飾的アニメーションの Animation 層分離] | §5.7 要求レベル |
| §5.7 | SHOULD [2 ガード原則の実装] | §5.7 要求レベル |
| §5.7 | SHOULD [translate / rotate / scale を含む機能的トランジションのガード] | §5.7 要求レベル |
| §5.7 | SHOULD [@keyframes 名と data-* 属性値の一致] | §5.7 要求レベル |
| §5.8 | SHOULD [Utility 層の責任限定] | §5.8 要求レベル |
| §5.8 | SHOULD NOT [Block 帰属スタイルの Utility 記述禁止] | §5.8 要求レベル |
| §6 | SHOULD [層識別プレフィックスの使用] | §6 要求レベル |
| §6 | SHOULD NOT [Element 連鎖の回避] | §6 要求レベル |
| §7 | SHOULD [セマンティック変数経由の参照] | §7 要求レベル |
| §7 | SHOULD [公開 API 変数の命名規則遵守] | §7 要求レベル |
| §7 | SHOULD NOT [プリミティブ変数の直接参照禁止] | §7 要求レベル |
| §8 | SHOULD [ディレクトリ名の層名一致] | §8 要求レベル |
| §8 | SHOULD [1 Block = 1 ファイルの維持] | §8 要求レベル |
| §8 | SHOULD [層帰属の明確化] | §8 要求レベル |

### MAY / MAY NOT

| § | 一言サマリ | 全文参照 |
|---|---|---|
| §5.1 | MAY [テーマ切替の Token 完結] | §5.1 要求レベル |
| §5.2 | MAY [Reset 層の使用任意] | §5.2 要求レベル |
| §5.7 | MAY [機能的トランジションの所属層への記述] | §5.7 要求レベル |
| §5.8 | MAY [セマンティックなファイルグループ化] | §5.8 要求レベル |
| §7 | MAY [グローバルトークンの直接参照] | §7 要求レベル |
| §7 | MAY [プライベートカスタムプロパティの定義] | §7 要求レベル |
