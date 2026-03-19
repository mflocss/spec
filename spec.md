# mFLOCSS 仕様 v1.1

> **ステータス**: ドラフト
> **最終更新**: 2026-03-17
> **著者**: shunei

---

## 1. Overview

### 1.1 Introduction

*This section is informative.*

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークである。

「どの層に、なぜ書くか」という問いに対し、明確な判断基準を提供する。`@layer` ベースの 8 層フラットアーキテクチャを採用する。

#### 対象範囲

- フレームワーク非依存（素の CSS を対象とする）
- プリプロセッサを前提としない（Sass 等との併用は妨げない）
- JavaScript フレームワーク固有の CSS-in-JS は対象外

### 1.2 不変原則

*This section is normative.*

mFLOCSS は以下の 4 原則に基づく。層数が変わってもこれらは変わらない。

1. **関心の分離** — 異なる問いに答えるスタイルは異なる層に分離する
2. **@layer による構造的制御** — 詳細度の問題を命名規則ではなくブラウザの仕組みで解決する
3. **判断基準の明示** — 各層に「何を書くか」だけでなく「なぜその層か」の基準がある
4. **CSS の進化への追従** — 層構成は CSS の進化に応じて適応させる設計余地を持つ（具体的な検討事項は README を参照）

### 1.3 設計制約

*This section is normative.*

不変原則を支える構造的制約。層数が変わっても維持される。

1. **一方向依存** — 上位層（番号が大きい層）は下位層を参照してよいが、逆方向の参照は禁止する。詳細ルールは §3 を参照

---

## 2. Conformance

*This section is normative.*

### 要求レベル

本仕様のキーワード MUST / MUST NOT / SHOULD / SHOULD NOT / MAY は RFC 2119 [RFC2119] に基づく。

| キーワード | 意味 |
|---|---|
| MUST | 絶対的な要求。違反は非準拠となる |
| MUST NOT | 絶対的な禁止。違反は非準拠となる |
| SHOULD | 推奨。正当な理由がある場合に限り逸脱を許容する |
| SHOULD NOT | 非推奨。正当な理由がある場合に限り逸脱を許容する |
| MAY | 任意。プロジェクトの判断に委ねる |

### 準拠条件

mFLOCSS v1.1 に準拠するとは、本仕様の全 MUST / MUST NOT ルールに違反しないことを意味する。設計判断を伴うルール（Portability Test 等）の判定は、本仕様が提供するテスト（§3 Portability Test, Layout Test, Responsibility Test）に基づく。

### バージョニング

- 層の追加・削除・統合はメジャーバージョン変更とする
- 既存層内のルール追加・変更はマイナーバージョン変更とする

---

## 3. Layer Architecture

*This section is normative.*

mFLOCSS は 8 つの層で構成される。

| 順序 | 層名 | プレフィックス | 責任 |
|---|---|---|---|
| 1 | tokens | — | プリミティブなデザイン値の定義 |
| 2 | theme | — | Tokens をセマンティック変数にマッピング |
| 3 | foundation | — | ブラウザ環境の初期化と基本スタイル |
| 4 | layout | `l-` | 位置と空間の配置 |
| 5 | component | `c-` | 配置先に左右されない再利用可能なパーツ |
| 6 | project | `p-` | サイト固有のパーツとデザイン要件 |
| 7 | animation | `a-` | 動きの分離管理 |
| 8 | utility | `u-` | 単一目的のスタイル上書き |

### 層間の依存方向

上位層（番号が大きい層）は下位層を参照してよい。逆方向の参照は禁止する。

- MUST NOT: 下位層から上位層のクラスやカスタムプロパティを参照してはならない
- 例: Component 層が Project 層のクラスに依存してはならない

依存方向ルールは **CSS の参照方向** に適用される。HTML の入れ子構造はこのルールの対象外である。Component の中に Project を配置し、その中に Component を置く構造（例: `.c-modal` > `.p-login-form` > `.c-button`）は、CSS で他の層のクラスを参照しない限り違反にならない。

### Portability Test

Component 層と Project 層の境界を判定する基準テスト。

> 「そのパーツを別のサイトにそのまま持っていけるか？」

- Yes → Component
- No → Project

### Responsibility Test

Component と Project の境界を補助的に判定するテスト。Portability Test が明確に Yes または No を返す場合、Responsibility Test の結果に関わらずその判定に従う。Portability Test で判断が曖昧な場合にのみ Responsibility Test を使用する。

> 「このスタイルは、パーツ自身の視覚的責任か？それとも使う側のデザイン要件か？」

- Yes（パーツ自身の責任）→ Component: ボタンの背景色、カードの角丸
- No（使う側の要件）→ Project: ヒーロー内のボタンのサイズ変更、カードの配置間隔

### Layer Judgment Flowchart

> **注記（Informative）**: このフローチャートは層判断の参考ガイドであり、規範的ルールではない。

スタイルをどの層に書くべきかを 6 ステップで判断する。

```
Step 1: デザイントークン（色の値、フォント名、z-index 値等）か？
  ├─ プリミティブな値の定義 → Tokens
  └─ セマンティックなマッピング → Theme

Step 2: ブラウザ環境の初期化・要素の基本スタイルか？
  └─ Yes → Foundation

Step 3: 配置と空間だけか？（色・文字・装飾に触れないか？）
  └─ Yes → Layout

Step 4: 別のサイトにそのまま持っていけるか？（Portability Test）
  ├─ Yes → Component
  └─ No → Project
  ※ Portability Test で判断が曖昧な場合は、Responsibility Test（§3）を補助的に使用する。

※ Step 5-6 は Step 1-4 の判定結果とは独立して評価する。
  1つのスタイルが Step 4 で Component と判定され、かつ Step 5 で
  Animation にも該当する場合、動きの部分を Animation 層に分離する。

Step 5: prefers-reduced-motion で無効化すべき動きか？
  └─ Yes → Animation

Step 6: 局所的な単一目的の微調整か？
  └─ Yes → Utility
```

#### よくある誤りパターン

**A. 機能近接バイアス**: 「テーブルのスクロールラッパー → Component」と判断する誤り。スクロール制御は使う側のコンテナの制約であり、テーブルパーツ自体の責任ではない。正しくは Project の Element。

**B. Layout への過干渉**: `l-section` に `text-align: center` を追加する誤り。テキスト整列は視覚的プロパティであり、Layout の責任（配置と空間）を超えている。正しくは Project。

**C. ラッパーの帰属誤り**: `overflow-x: auto` を Component のラッパーとして定義する誤り。コンテナ幅の制約対応はサイト固有のデザイン要件に依存する。正しくは `p-xxx__table-wrap` として Project の Element に持たせる。

---

## 4. Layer Order Declaration

*This section is normative.*

### 先制宣言

MUST: CSS Cascading and Inheritance Level 5 [CSS-CASCADE-5] に定義される `@layer` の優先順位宣言をエントリポイント CSS の先頭で、全ての `@import` に先行して行わなければならない。

> **Example:**

```css
/* layer-order.css */
@layer tokens, theme, foundation, layout, component, project, animation, utility;
```

### @layer と @property

MUST: `@property` を使用する場合は `@layer` の外に配置しなければならない。現行の CSS 仕様（CSS Properties and Values API Level 1）において、`@layer` 内の `@property` は無視される。`@property` 自体の使用は任意である。

### !important の優先度逆転

`@layer` 内で `!important` を使用した場合、通常とは逆順で優先される（先に宣言された層が勝つ）。MUST NOT: Utility 層以外の全層で `!important` を使用してはならない。この制約により優先度逆転の複雑性を回避する。

### 外部 CSS の層配置

`@layer` に属さない CSS（unlayered CSS）は全ての layered CSS より優先される。外部ライブラリの CSS を `<link>` タグ等で直接読み込むと、mFLOCSS の層構造を貫通し、意図しない上書きが発生する。

MUST: 外部 CSS は `@import url() layer()` または npm + バンドラーを使用し、いずれかの層に取り込まなければならない。

外部 CSS の層配置は、自作 CSS と同じ判断基準で決定する。出自（外部 / 自作）による特別扱いは行わない。

| 外部 CSS の種類 | 配置先 |
|---|---|
| リセット系ライブラリ | `layer(foundation)` |
| UI コンポーネント系ライブラリ | `layer(component)` |

---

## 5. Layer Definitions

*This section is normative.*

### 5.1 Tokens

**責任**: プリミティブなデザイン値の定義

- MUST: `:root` セレクタのみを使用しなければならない
- MUST: 値のみを定義しなければならない（セマンティクスを持たせない）
- MUST NOT: 他の層のカスタムプロパティを参照してはならない

**トークンの分類**:

Tokens はブランドトークンとグローバルトークンに分類される。

| 分類 | 性質 | 例 | Theme 経由 |
|------|------|-----|-----------|
| ブランドトークン | プロジェクトごとに変わる値 | color, typography, structure | MUST |
| グローバルトークン | プロジェクト非依存の普遍値 | ease, z-index, font-weight | 直接参照 MAY |

判断基準: 「ブランドやプロジェクトが変わったとき、この値を変更するか？」— Yes ならブランドトークン、No ならグローバルトークン。

**命名規則**: `--{カテゴリ}-{名前}`

> **Example:**

```css
@layer tokens {
  :root {
    --slate-600: oklch(0.45 0.02 260);
    --font-ja: "Noto Sans JP", sans-serif;
    --ease-out-cubic: cubic-bezier(0.33, 1, 0.68, 1);
    --z-header: 100;
  }
}
```

SHOULD: カテゴリ別にファイルを分割する（color / typography / structure / ease / z）。

### 5.2 Theme

**責任**: Tokens をセマンティック変数にマッピング

- MUST: Tokens のカスタムプロパティを参照しなければならない（ハードコード値の直接指定は禁止）
- MUST: ダークモード / テーマ切替はこの層のみで完結させなければならない

SHOULD: `light-dark()` 関数を使用する。

**命名規則**: `--color-{役割}`, `--font-{役割}`

> **Example:**

```css
@layer theme {
  :root {
    color-scheme: light dark;
    --color-main: light-dark(var(--slate-900), var(--slate-100));
    --color-surface: light-dark(var(--white), var(--slate-900));
    --font-body: var(--font-ja);
  }
}
```

### 5.3 Foundation

**責任**: ブラウザ環境の初期化と基本スタイル

- MUST: 要素型セレクタのみを使用しなければならない（クラスセレクタ・ID セレクタ禁止）。SHOULD: 属性セレクタ、擬似クラス、擬似要素は、`:where()` 内で要素型セレクタと組み合わせて使用すべきである
- SHOULD: `:where()` で詳細度をゼロに保つべきである。アタッチメント方式で取り込む外部リセット CSS は `:where()` を使用していない場合がある。`@layer` による層順序制御が詳細度の予測可能性を構造的に担保するため、`:where()` は推奨とする
- MUST: ブランドトークンについては Theme のセマンティック変数を参照しなければならない（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）

SHOULD: Foundation 層はサブレイヤー `foundation.reset`（外部リセット CSS）と `foundation.base`（自作ベーススタイル）に分離すべきである。これにより、外部リセット CSS が `:where()` を使用していない場合でも、自作ベーススタイルがリセットを上書きできる。

サブレイヤーを使用する場合（SHOULD）:

> **Example:**

```css
/* foundation/index.css */
@layer reset, base;

/* foundation/reset.css — 外部リセット CSS */
@import url("modern-normalize.css") layer(reset);

/* foundation/base.css — 自作ベーススタイル */
@layer base {
  :where(body) {
    line-height: 1.5;
  }
}
```

SHOULD: reset / base / form の 3 ファイルに分割する。form を独立させる理由は、フォーム要素（input, select, textarea, button）はブラウザ間のデフォルトスタイル差異が最も大きく、正規化のコード量が多くなるためである。

> **注記（Informative）**: リセット CSS はアタッチメント方式とする。リファレンス実装が参考を提供するが、各プロジェクトに適したリセットを選定・適用すること。リセット CSS の内部実装は本仕様の準拠対象外とする。

サブレイヤーを使用しない場合:

> **Example:**

```css
@layer foundation {
  :where(body) {
    font-family: var(--font-body);
    color: var(--color-main);
    background-color: var(--color-surface);
  }
}
```

### 5.4 Layout

**責任**: 位置と空間の配置

**プレフィックス**: `l-`

- MUST NOT: 見た目に関するプロパティ（`color`, `font-size`, `background-color`, `border`, `text-align` 等の視覚的プロパティ）を宣言してはならない

SHOULD: `container-type: inline-size` を宣言し、Container Queries の基盤を提供する。Container Queries を使用しないプロジェクトではこの限りではない。

MAY: `container-name` を併用し、名前付きコンテナを定義してよい。セクション単位で名前付きコンテナを定義すると、`@container` での参照先を明示できる。

#### Container Queries の層責任

> **注記（Informative）**: Container Queries は複数層にまたがる仕組みであるため、層責任の全体像をここにまとめて示す。各層の個別ルールは該当セクション（§5.5, §5.6）に従う。

| 責任 | 層 |
|---|---|
| `container-type` の宣言 | Layout |
| `container-name` の定義 | Layout（MAY） |
| `@container` によるスタイル切替 | Component / Project |

プライベートカスタムプロパティ（`--_xxx`）の使用については §7 Custom Properties を参照。

**検証問い（Layout Test）**: 「このスタイルを変えると、中身の見た目（色・文字・装飾）が変わるか？」 — Yes なら Layout ではない。

> **Example:**

```css
@layer layout {
  .l-section {
    --_padding-min: 3.75rem;
    --_padding-max: 6.25rem;

    container-type: inline-size;
    container-name: section;
    padding-block: clamp(var(--_padding-min), 8vi, var(--_padding-max));
  }
}
```

### 5.5 Component

**責任**: 配置先に左右されない再利用可能なパーツ

**プレフィックス**: `c-`

- MUST: Portability Test に合格しなければならない（「別サイトにそのまま持っていけるか？」）
- MUST NOT: 特定のサイトでしか使えない見た目にしてはならない
- MUST: ブランドトークンについては Theme のセマンティック変数のみを参照しなければならない（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）
- MUST NOT: 外部レイアウトに影響するプロパティ（ルート要素の `margin`, `position: fixed/sticky`, ルート要素の `overflow` 等）を Component のルート要素に含めてはならない — 配置は使う側の責任（Responsibility Test）である

> **注記（Informative）**: Component 内部の Element（`__element`）間の余白（`margin`, `gap`）や内部配置（`position: relative` / `absolute`）は上記 MUST NOT の対象外である。これらは Component 自身の視覚的責任に該当する

SHOULD: 1 コンポーネント 1 ファイルとする。

MAY: Component の中に他の Component を内包してよい（例: `.c-card` の中に `.c-button` を配置する）。内包しても Portability Test に合格するなら Component のままである。

SHOULD: Component と Project の判断に迷った場合は Component とする。Project で上書き可能だが、逆（Project → Component への汎化）は困難であるため。Portability Test で明確に No と判断できる場合はこの限りではない。

### 5.6 Project

**責任**: サイト固有のパーツとデザイン要件

**プレフィックス**: `p-`

- Component や Layout を内包し、ページやセクションに合わせた配置・装飾を担う
- インラインスタイルについては §7 Custom Properties を参照

SHOULD: ページ単位または機能単位でファイルを分割する。

#### 上書きパターン

Project 層は上位層として、Component のスタイルをページやセクションに合わせて上書きできる。

| パターン | 用途 | 例 |
|---|---|---|
| 直接プロパティ上書き | 特定の配置先でパーツのスタイルを変更 | `.p-hero > .c-button { font-size: ... }` |
| CSS 変数経由 | Component/Layout が公開するプライベートカスタムプロパティの値を設定 | `.p-about { --_padding-min: 2.5rem; }` |
| Modifier（Component 自身） | 汎用バリエーション | `.c-button.-primary` |

SHOULD: 直接プロパティ上書きと CSS 変数経由のどちらも使用できる場合は、CSS 変数経由を優先する。CSS 変数経由は Component/Layout の内部構造に依存しないため保守性が高い。

Modifier は Component に内包される再利用可能なバリエーション。Project 上書きはサイト固有のデザイン要件に基づき Component のスタイルを調整するもの。

Layout の振る舞いをページやセクションに合わせて変えたい場合は、Project の Block または Element として定義する（例: `class="l-section p-about"`, `class="l-inner p-about__inner"`）。

> **Example:**

```css
@layer project {
  /* Project の Block として Layout の配置先適応を担う */
  .p-about {
    --_padding-min: 2.5rem;
    --_padding-max: 3.75rem;
  }

  /* Project 固有のパーツ */
  .p-hero__lead {
    text-align: center;
    margin-block-start: 2.5rem;
  }

  /* Project の Element としてレイアウトの配置先適応を担う */
  .p-about__inner {
    max-inline-size: 50rem;
  }

  /* テーブルのスクロールラッパー（サイト固有の制約対応） */
  .p-philosophy__table-wrap {
    overflow-x: auto;
  }
}
```

### 5.7 Animation

**責任**: 動きの分離管理

**プレフィックス**: `a-`

- MUST: Animation 層のスタイルは、`prefers-reduced-motion: reduce` 環境でアニメーションが無効になり、`scripting: none` 環境で要素が不可視にならない実装としなければならない

この 2 条件の考慮を **2 ガード原則** と呼ぶ。

SHOULD: `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガードを使用する。条件を満たさない場合にブロック全体が不適用になり、フォールバック安全性が高い。

上記の統合ガードパターン（SHOULD）に従えば、この MUST は自動的に満たされる。統合ガードを使用しない場合は、`prefers-reduced-motion: reduce` でアニメーション関連プロパティが初期値に解決されること、`scripting: none` で要素の可視性が維持されることを個別に検証する。

> **Example:**

```css
@layer animation {
  @media (prefers-reduced-motion: no-preference) and (scripting: enabled) {
    .a-fade-in {
      opacity: 0;
      transition: opacity 0.4s var(--ease-out-cubic);
    }
    .a-fade-in.is-active {
      opacity: 1;
    }
  }
}
```

### 5.8 Utility

**責任**: 単一目的のスタイル上書き

**プレフィックス**: `u-`

- MUST: `!important` を付与しなければならない
- MUST NOT: 特定の Block や Element に帰属できるスタイルを Utility に書いてはならない — Utility は特定の Block に帰属しない横断的かつ局所的な単一目的のスタイルに限る
- MUST NOT: Utility 層以外の全層で `!important` を使用してはならない

**適切な用途**: アクセシビリティ非表示（`u-visually-hidden`）、レスポンシブ表示制御

**不適切な用途**: 色の定義、レイアウトの組み立て、コンポーネントの装飾

**ファイル構成**: SHOULD: セマンティックな意味でグループ化する。1 クラス 1 ファイルではなく、関連するユーティリティをまとめる（例: `u-hidden.css` に `u-visually-hidden` と `u-hidden-sp` をまとめる）。Utility は最小限に留まるため、ファイル数も最小で済む。

> **Example:**

```css
@layer utility {
  .u-visually-hidden {
    position: absolute !important;
    inline-size: 1px !important;
    block-size: 1px !important;
    padding: 0 !important;
    margin: -1px !important;
    overflow: hidden !important;
    clip-path: inset(50%) !important;
    white-space: nowrap !important;
    border: 0 !important;
  }
}
```

---

## 6. Naming Conventions

*This section is normative.*

### クラス名

```
.{prefix}-{name}__{element}
.{prefix}-{name}.-modifier
```

| 要素 | 形式 | 例 |
|---|---|---|
| Block | `.{prefix}-{name}` | `.c-button`, `.l-section` |
| Element | `.__{element}` | `.c-card__title` |
| Modifier | `.-{modifier}` | `.c-button.-primary` |
| State | `.is-{state}`, `.has-{state}` | `.is-active`, `.has-children` |

### Modifier と State の使い分け

- **Modifier（`.-xxx`）**: 静的なバリエーション。HTML に記述し、原則として変化しない
- **State（`.is-xxx`, `.has-xxx`）**: 動的な状態。JS やユーザー操作により変化する

State のスタイルは、対象の Block が属する層に記述する（例: `.c-button.is-active` は Component 層、`.a-fade-in.is-active` は Animation 層）。

### Modifier に `-` を採用する理由

BEM の `--` ではなく `.-modifier` を採用する。

- HTML がコンパクトになる（`class="c-button -primary"` vs `class="c-button c-button--primary"`）
- CSS Nesting との相性が良い

### Element の深さ

SHOULD NOT: Element を 2 階層以上ネストすべきではない（`__element1__element2` は非推奨）。Element が深くなる場合はコンポーネントの分離を検討する。

### JS 連携

`js-` プレフィックスは設けない。JS からの要素取得には `data-*` 属性または ARIA 属性を使用する。

### ファイル名

MUST: ファイル名は主要クラス名と一致させなければならない — `{prefix}-{name}.css`。1 ファイルに関連クラスをグループ化する場合は、代表クラス名をファイル名とする（例: `u-hidden.css`）。

例: `.c-button` → `c-button.css`, `.l-section` → `l-section.css`

### カスタムプロパティ命名まとめ

| 層 | パターン | 例 |
|---|---|---|
| Tokens | `--{カテゴリ}-{名前}` | `--slate-600`, `--font-ja` |
| Theme | `--color-{役割}`, `--font-{役割}` | `--color-main`, `--font-body` |
| Private | `--_{名前}` | `--_min`, `--_max` |

---

## 7. Custom Properties

*This section is normative.*

### 3 層参照チェーン

カスタムプロパティは以下の 3 層を経由して使用する。

```
Tokens（値）→ Theme（意味）→ Foundation 以降（使用）
```

- MUST: Foundation 以降の層はブランドトークンについて Theme のセマンティック変数を参照しなければならない
- MUST NOT: Foundation 以降の層がブランドトークンを直接参照してはならない（Theme を経由する）
- MAY: グローバルトークン（ease, z-index 等）は Foundation 以降の層から直接参照してよい（Tokens 層のトークン分類を参照）

例外: Tokens の値を Theme 層でマッピングする際のみ、ブランドトークンへの直接参照が許される。

### プライベートカスタムプロパティパターン

MAY: Layout 以降の層（Layout, Component, Project, Animation）でプライベートカスタムプロパティ（`--_xxx`）を定義してよい。`--_` プレフィックスは内部変数であることを命名で明示する。外部 API ではないことを示す。プライベートカスタムプロパティは外部 API ではないが、上位層（Project 等）から値を設定することは許容される。これは §5.6 の CSS 変数経由上書きパターンの基盤となる。

> **Example:**

```css
/* Layout 層で定義 */
@layer layout {
  .l-section {
    --_padding-min: 3.75rem;
    --_padding-max: 6.25rem;

    padding-block: clamp(var(--_padding-min), 8vi, var(--_padding-max));
  }
}
```

Layout の振る舞いをページやセクションに合わせて変えたい場合は、Project の Block または Element として定義する。

> **Example:**

```css
@layer project {
  /* Project の Block として l-section のプライベートカスタムプロパティを上書き */
  .p-about {
    --_padding-min: 2.5rem;
    --_padding-max: 3.75rem;
  }

  /* Project の Element として l-inner の振る舞いを配置先適応 */
  .p-about__inner {
    max-inline-size: 50rem;
  }
}
```

### インラインスタイル

MUST NOT: HTML マークアップに静的なインラインスタイルを記述してはならない。インラインスタイルはどの `@layer` にも属さず、mFLOCSS の「どこに何を書くか」を追跡可能にする設計思想と矛盾する。

MAY: JS からプライベートカスタムプロパティの値を動的に注入してよい。CSS が「何をするか」を定義し、JS が「いつ・どの値で」を決める疎結合パターンとして許容する。

---

## 8. File Architecture

*This section is normative.*

### ディレクトリ構造

MUST: ディレクトリ名は層名と一致させなければならない。

```
css/
├── layer-order.css
├── style.css
├── tokens/
├── theme/
├── foundation/
├── layout/
├── component/
├── project/
├── animation/
└── utility/
```

### layer-order.css

`@layer` の先制宣言のみを含む。スタイル定義は一切置かない。

### style.css（エントリポイント）

MUST: `layer-order.css` を最初に読み込み、その後層順にファイルをインポートしなければならない。

SHOULD: 各ファイルがどの層に属するかを明確にする。方法はプロジェクトの規模やツールチェーンに応じて選択してよい。

| 方式 | 説明 |
|---|---|
| `@import layer()` | エントリポイントで層の帰属を一元管理する（推奨） |
| 層ごとの index.css | 各層の index.css 内で `@layer` ブロックを使ってまとめる |
| ビルドツール自動解決 | ビルドツールが glob import で自動的にファイルを収集する |

いずれの方式でも、ディレクトリ名・ファイルプレフィックス・クラスプレフィックスによって層の帰属は識別できる。

---

## 9. Responsive Strategy

*This section is normative.*

### アプローチの選択

SP ファースト / PC ファーストはプロジェクトごとに判断する。本仕様はどちらも許容する。

ブレークポイントもプロジェクトごとに設定する。768px / 1024px は参考値であり、本仕様の規定値ではない。

### 3 つの手法と使い分け

| 手法 | 用途 | 適用例 |
|---|---|---|
| Container Queries | コンポーネント単位のレスポンシブ | カードの画像/コンテンツの縦横切替、テーブルのレイアウト変更 |
| Media Queries | ビューポート全体の離散的変化 | ナビゲーション切替、カラム数変更 |
| `clamp()` | 連続的な流体デザイン | フォントサイズ、余白の滑らかな変化 |

SHOULD: コンポーネント単位のレスポンシブデザインには Container Queries を使用する。ビューポート全体の変化には Media Queries、連続的な流体デザインには `clamp()` を使い分ける。

Container Queries の層責任（`container-type` / `container-name` / `@container` の配置先）は §5.4 Layout を参照。

### Container Queries の単位

SHOULD: Container Queries 内のサイズ指定には `cqi`（container query inline-size）を使用する。`vw` はビューポート基準であり、コンテナ基準の設計と矛盾する。

> **Example:**

```css
@container (inline-size >= 40em) {
  .c-card {
    font-size: clamp(0.875rem, 2cqi, 1.125rem);
  }
}
```

### メディアクエリの種別

本仕様が関与するメディア特性:

| メディア特性 | 用途 |
|---|---|
| `scripting` | JS 有効/無効の判定（2 ガード原則） |
| `prefers-reduced-motion` | モーション軽減の判定（2 ガード原則） |

---

## Appendix A: Glossary

*This section is normative.*

| 用語 | 定義 |
|---|---|
| **Portability Test** | 「そのパーツを別のサイトにそのまま持っていけるか？」— Component と Project の境界を判定する基準テスト（初出: §3） |
| **Layout Test** | 「このスタイルを変えると、中身の見た目（色・文字・装飾）が変わるか？」— Layout 層の責任範囲を判定する検証問い（初出: §5.4） |
| **Responsibility Test** | 「このスタイルは、パーツ自身の視覚的責任か？それとも使う側のデザイン要件か？」— Component と Project の境界を補助的に判定するテスト。Portability Test が明確な判定を返す場合はそちらに従う（§3 参照）（初出: §3） |
| **ブランドトークン** | プロジェクトごとに変わるデザイン値（color, typography, structure）。Foundation 以降の層では Theme 経由で参照しなければならない（初出: §5.1） |
| **グローバルトークン** | プロジェクト非依存の普遍値（ease, z-index, font-weight）。Foundation 以降の層から直接参照してよい（初出: §5.1） |
| **3 層参照チェーン** | Tokens（値）→ Theme（意味）→ Foundation 以降（使用）。カスタムプロパティの参照パスを規定する（初出: §7） |
| **プライベートカスタムプロパティ** | `--_` プレフィックスを持つ変数。内部 API であり、外部からの直接依存を想定しない（初出: §5.4） |
| **先制宣言** | `layer-order.css` における `@layer` の優先順位宣言。全スタイル定義に先行して記述される（初出: §4） |
| **2 ガード原則** | Animation 層で `prefers-reduced-motion` と `scripting` の 2 条件を考慮すること。推奨は `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガード（初出: §5.7） |

---

## Appendix B: References

### Normative References

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
- **[CSS-CASCADE-5]** Atkins Jr., T.; Rivoal, F.; Lilley, C., "CSS Cascading and Inheritance Level 5", W3C Candidate Recommendation.

### Informative References

- **[FLOCSS]** Hiloki, "FLOCSS — Foundation Layout Object CSS".
- **[BOOK]** shunei,『そのFLOCSS、なぜそこに書いた？』.

---

## Appendix C: Changes

*This section is informative.*

### v1.0 → v1.1 の変更点

#### 主要な変更

- §1 のサブセクション化: Introduction [Informative], 不変原則 [Normative], 設計制約 [Normative] に分離
- §1.3 設計制約の新設: 一方向依存ルールを不変原則を支える構造的制約として規定
- §3 Responsibility Test の追加: Portability Test を補助する判定テスト
- §3 Layer Judgment Flowchart の本文統合: 旧 Appendix A を §3 に移動し、誤りパターンを追加
- §3 依存方向: CSS の参照方向に限定し、HTML の入れ子構造はルール対象外と明確化
- §4 外部 CSS の層配置セクションの追加
- §4 CSS Cascading and Inheritance Level 5 への明示的参照の追加
- §5.1 トークン分類（ブランドトークン / グローバルトークン）の導入
- §5.4 Container Queries の層責任テーブルを §9 から移動・統合（container-name ガイダンスを追加）
- §5.4 に container-name の MAY ルールを新設
- §5.5 「迷ったら Component」の SHOULD 化と条件付き限定
- §5.6 上書きパターンの拡充（CSS 変数経由パターンの追加）
- §5.7 統合ガードの SHOULD 化と説明の簡潔化
- §5.8 u-visually-hidden の論理プロパティ対応
- §7 プライベートカスタムプロパティの上位層設定許容の明確化
- §8 ディレクトリ構造の簡素化（ファイル例の削除）
- Normative / Informative ラベルの付与（W3C パターン）
- Example マーキングの付与
- References セクションの新設（Appendix B）
- Glossary に各用語の初出セクション番号を付記
- Glossary: Responsibility Test の定義文を更新（Portability Test との優先関係を明記）
- §2 準拠条件: 設計判断を伴うルールの判定基準（テスト参照）を追記
- §5.3 Foundation: 「要素セレクタのみ」を「要素型セレクタのみ」に厳密化し、属性セレクタ・擬似クラス・擬似要素の許容範囲を明記
- §5.3 Foundation の `:where()` 詳細度ゼロを MUST から SHOULD に変更（外部リセット CSS のアタッチメント方式を考慮）
- §5.5 Component: MUST NOT の例外（内部 Element）を構造的に分離
- §5.5 Component: サイト固有禁止ルールの MUST → MUST NOT 修正（RFC 2119 準拠）
- §5.7 Animation: 2 ガード原則の MUST を達成条件ベースに書き直し
- §5.8 Utility: MUST NOT の判定基準を「Block/Element への帰属可否」に明確化
- §9 Container Queries を SHOULD で規定
- §5.3 Foundation にサブレイヤー構成（foundation.reset / foundation.base）を SHOULD で追加

#### 削除された内容

- 書籍参照（`→ 書籍 chX`）を本文から削除（Appendix B の Informative References に集約）
- カラートークンの階層化参考（§5.1）
- style.css のインポート例と注記（§8）
- コンポーネント追加/削除手順（§8）
- 将来の展望セクション（§1）

#### 章番号対応表

| v1.0 | v1.1 | 内容 |
|---|---|---|
| §1 Introduction | §1 Overview | 親セクション名を変更。§1.1/§1.2/§1.3 にサブセクション化 |
| — | §1.1 Introduction [Informative] | 旧 §1 の導入部分 |
| — | §1.2 不変原則 [Normative] | 旧 §1 の不変原則部分 |
| — | §1.3 設計制約 [Normative] | v1.1 で新設 |
| §1 不変原則（小見出し） | §1.2 不変原則 | サブセクションに昇格 |
| §2 Conformance | §2 Conformance | 変更なし |
| §3 Layer Architecture | §3 Layer Architecture | Responsibility Test, Flowchart, 誤りパターン追加 |
| §4 Layer Order Declaration | §4 Layer Order Declaration | 外部 CSS の層配置追加 |
| §5 Layer Definitions | §5 Layer Definitions | 各層の強化 |
| §6 Naming Conventions | §6 Naming Conventions | 変更なし |
| §7 Custom Properties | §7 Custom Properties | 上位層設定許容の明確化 |
| §8 File Architecture | §8 File Architecture | 簡素化 |
| §9 Responsive Strategy | §9 Responsive Strategy | Container Queries テーブルを §5.4 に統合 |
| §10 Appendix A: Flowchart | — | §3 に統合、削除 |
| §11 Appendix B: Glossary | Appendix A: Glossary | 繰り上げ、初出番号付記 |
| — | Appendix B: References | 新設 |
| — | Appendix C: Changes | 新設 |
