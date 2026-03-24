# mFLOCSS 仕様 v1.1

> **ステータス**: ドラフト
> **最終更新**: 2026-03-23
> **著者**: shunei

---

## 1. Overview

### 1.1 Introduction

*This section is informative.*

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークである。

「どの層に、なぜ書くか」という問いに対し、明確な判断基準を提供する。`@layer` ベースの 8 層フラットアーキテクチャを採用する。

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークであり、ルールブックではない。本仕様はその判断基準を厳密に定義する。MUST は設計の構造的整合性を保証するルール（`@layer` による層順序の固定・層の分離・命名規則の一貫性等）に適用し、設計判断の推奨は SHOULD で表現する。

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

1. **一方向依存** — 上位層（番号が大きい層）は下位層を参照してよいが、逆方向の参照は禁止する（MUST NOT、§3 参照）。詳細ルールは §3 を参照

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

mFLOCSS v1.1 に準拠するとは、本仕様の全 MUST / MUST NOT ルールに違反しないことを意味する。MUST は `@layer` の構造的整合性・アクセシビリティ・命名体系の維持に限定される。設計判断の品質（層の選択・トークン参照チェーン等）は SHOULD で推奨し、遵守するほど設計の一貫性と保守性が向上する。

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
| 7 | animation | — | 動きの分離管理 |
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

Step 5: 装飾的アニメーション（視覚演出）か？
  ├─ Yes → Animation（2 ガード原則を適用）
  └─ 機能的トランジション（インタラクションフィードバック）→ Component / Project に残す

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

MUST: `@property` を使用する場合は `@layer` の外に配置しなければならない。現行の CSS 仕様 [CSS-PROPERTIES-VALUES-1] において、`@layer` 内の `@property` は無視される。`@property` 自体の使用は任意である。

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
- SHOULD: 値のみを定義すべきである（セマンティクスを持たせない）
- MUST NOT: 他の層のカスタムプロパティを参照してはならない

**トークンの分類**:

Tokens はブランドトークンとグローバルトークンに分類される。

| 分類 | 性質 | 例 | Theme 経由 |
|------|------|-----|-----------|
| ブランドトークン | プロジェクトごとに変わる値 | color, typography, structure | SHOULD |
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

SHOULD: カテゴリ別にファイルを分割すべきである（color / typography / structure / ease / z）。

### 5.2 Theme

**責任**: Tokens をセマンティック変数にマッピング

- SHOULD: Tokens のカスタムプロパティを参照すべきである（ハードコード値の直接指定は非推奨）
- SHOULD: ダークモード / テーマ切替はこの層のみで完結させるべきである
- SHOULD: `:root` セレクタを基本とすべきである。MAY: ダークモード切替で `data` 属性セレクタ（例: `[data-theme='dark']`）を使用してよい。`light-dark()` 関数を使用する場合（SHOULD）は `:root` のみで完結する

SHOULD: `light-dark()` 関数を使用すべきである。

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

**検証問い**: 「このスタイルは、要素のブラウザデフォルトの正規化または基本スタイルの定義か？」 — Yes なら Foundation。No（特定のコンテキストに依存する）なら上位層。

- MUST: 要素型セレクタのみを使用しなければならない（クラスセレクタ・ID セレクタ禁止）。SHOULD: 属性セレクタ、擬似クラス、擬似要素は、`:where()` 内で要素型セレクタと組み合わせて使用すべきである
- SHOULD: `:where()` で詳細度をゼロに保つべきである。アタッチメント方式で取り込む外部リセット CSS は `:where()` を使用していない場合がある。`@layer` による層順序制御が詳細度の予測可能性を構造的に担保するため、`:where()` は推奨とする
- SHOULD: ブランドトークンについては Theme のセマンティック変数を参照すべきである（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）

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

SHOULD: reset / base / form の 3 ファイルに分割すべきである。form を独立させる理由は、フォーム要素（input, select, textarea, button）はブラウザ間のデフォルトスタイル差異が最も大きく、正規化のコード量が多くなるためである。

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

- SHOULD NOT: 見た目に関するプロパティ（`color`, `font-size`, `background-color`, `border`, `text-align` 等の視覚的プロパティ）を宣言すべきでない

SHOULD: `container-type: inline-size` を宣言し、Container Queries の基盤を提供すべきである。Container Queries を使用しないプロジェクトではこの限りではない。

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
    --section-padding-min: 3.75rem;
    --section-padding-max: 6.25rem;

    container-type: inline-size;
    container-name: section;
    padding-block: clamp(var(--section-padding-min), 8vi, var(--section-padding-max));
  }
}
```

### 5.5 Component

**責任**: 配置先に左右されない再利用可能なパーツ

**プレフィックス**: `c-`

- SHOULD: Portability Test に合格すべきである（「別サイトにそのまま持っていけるか？」）
- SHOULD NOT: 特定のサイトでしか使えない見た目にすべきでない
- SHOULD: ブランドトークンについては Theme のセマンティック変数のみを参照すべきである（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）
- SHOULD NOT: 外部レイアウトに影響するプロパティ（ルート要素の `margin`, `position: fixed/sticky`, ルート要素の `overflow` 等）を Component のルート要素に含めるべきでない — 配置は使う側の責任（Responsibility Test）である

> **注記（Informative）**: Component 内部の Element（`__element`）間の余白（`margin`, `gap`）や内部配置（`position: relative` / `absolute`）は上記 SHOULD NOT の対象外である。これらは Component 自身の視覚的責任に該当する

SHOULD: 1 コンポーネント 1 ファイルとすべきである。

MAY: Component の中に他の Component を内包してよい（例: `.c-card` の中に `.c-button` を配置する）。内包しても Portability Test に合格するなら Component のままである。

SHOULD: Component と Project の判断に迷った場合は Component とすべきである。Project で上書き可能だが、逆（Project → Component への汎化）は困難であるため。Portability Test で明確に No と判断できる場合はこの限りではない。

### 5.6 Project

**責任**: サイト固有のパーツとデザイン要件

**プレフィックス**: `p-`

- SHOULD NOT: Portability Test（§3）に合格するスタイルを Project 層に記述すべきでない。該当するスタイルは Component 層（§5.5）に記述する
- Component や Layout を内包し、ページやセクションに合わせた配置・装飾を担う

SHOULD: サイト固有のデザイン要件があるセクションには、セクションルートに Project Block を付与し、セクション内の要素は Element として構築すべきである。サイト固有のスタイリングが不要なセクションは Layout と Component のみで構成してよい。
- インラインスタイルについては §7 Custom Properties を参照

SHOULD: ページ単位または機能単位でファイルを分割すべきである。

#### 上書きパターン

Project 層は上位層として、Component のスタイルをページやセクションに合わせて上書きできる。

| パターン | 用途 | 例 |
|---|---|---|
| 直接プロパティ上書き | 特定の配置先でパーツのスタイルを変更 | `.p-hero > .c-button { font-size: ... }` |
| CSS 変数経由 | Component/Layout が公開する公開 API カスタムプロパティの値を設定 | `.p-about { --section-padding-min: 2.5rem; }` |
| Modifier（Component 層 / Project 層で定義） | 汎用バリエーション | `.c-button.-primary` |

SHOULD: 直接プロパティ上書きと CSS 変数経由のどちらも使用できる場合は、CSS 変数経由を優先すべきである。CSS 変数経由は Component/Layout の内部構造に依存しないため保守性が高い。

Modifier は Component に内包される再利用可能なバリエーション。Project 上書きはサイト固有のデザイン要件に基づき Component のスタイルを調整するもの。

Layout や Component の振る舞いをページやセクションに合わせて変えたい場合は、Project の Block または Element として定義する（例: `class="l-section p-about"`, `class="l-inner p-about__inner"`）。

MAY: Project が内包する Component や Layout の要素に、現時点で固有スタイルがなくても Project の Element クラスを付与してよい（例: `class="c-section-heading p-about__heading"`）。カスタマイズ時に CSS の追加だけで対応できる拡張点として機能する。

> **Example:**

```css
@layer project {
  /* Project の Block として Layout の配置先適応を担う */
  .p-about {
    --section-padding-min: 2.5rem;
    --section-padding-max: 3.75rem;
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

**セレクタ**: `data-animate` 属性および `data-stagger` 属性

- `[data-animate="{種別}"]` — 要素自体のアニメーション種別を指定する（例: `data-animate="fade-in"`, `data-animate="slide-up"`）
- `[data-stagger="{種別}"]` — 子要素に段階的な遅延を適用するコンテナ。値はアニメーション種別で、`data-animate` と同じ `@keyframes` を共有する（例: `data-stagger="fade-in-slide-up"`）。JS が各子要素に `--stagger-delay` を設定する

> **設計根拠**: Animation 層のスタイルは JS（IntersectionObserver 等）と連動して状態クラスを付与する。JS からの要素取得には `data-*` 属性を使用する（§6 JS 連携）ため、クラスプレフィックス（`a-`）ではなく `data-animate` 属性を採用する。

- MUST: Animation 層のスタイルは、`prefers-reduced-motion: reduce` 環境でアニメーションが無効になり、`scripting: none` 環境で要素が不可視にならない実装としなければならない

#### 動きの分類

Animation 層に分離すべき動きと、Component/Project に残してよい動きを区別する。

| 分類 | 性質 | 例 | 層 |
|---|---|---|---|
| 装飾的アニメーション | 視覚演出。なくても機能に影響しない | fade-in, slide-up, stagger, parallax | Animation |
| 機能的トランジション | インタラクションフィードバック。ユーザー操作に対する応答 | hover の色変化, ボタンの translate, フォーカスリングの遷移 | Component / Project |

SHOULD: 装飾的アニメーションは Animation 層に分離し、2 ガード原則を適用すべきである。機能的トランジションは、対象の Block が属する層（Component または Project）に記述してよい。

判断基準: 「その動きを無効化しても、インタラクションの意味が伝わるか？」 — Yes（なくても伝わる）→ 装飾的 → Animation。No（ないと操作感が損なわれる）→ 機能的 → Component/Project。

この 2 条件の考慮を **2 ガード原則** と呼ぶ。

SHOULD: `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガードを使用すべきである。条件を満たさない場合にブロック全体が不適用になり、フォールバック安全性が高い。

上記の統合ガードパターン（SHOULD）に従えば、この MUST は自動的に満たされる。統合ガードを使用しない場合は、`prefers-reduced-motion: reduce` でアニメーション関連プロパティが初期値に解決されること、`scripting: none` で要素の可視性が維持されることを個別に検証する。

#### @keyframes 命名

SHOULD: `@keyframes` を使用する場合、`@keyframes` 名は対応する `data-*` 属性の値と一致させるべきである（例: `data-animate="scale-in"` → `@keyframes scale-in`）。`data-animate` と `data-stagger` は同じ種別値を持つ場合、同じ `@keyframes` を共有する。`@layer` は `@keyframes` 名のスコープを分離しないため、属性値との対応関係を明確にすることで名前衝突のリスクを低減する。

> **Example:**

```css
@layer animation {
  @media (prefers-reduced-motion: no-preference) and (scripting: enabled) {
    /* transition ベース（@keyframes 不要） */
    [data-animate="fade-in"] {
      opacity: 0;
      transition: opacity 0.4s var(--ease-out-cubic);
    }
    [data-animate="fade-in"].is-active {
      opacity: 1;
    }

    /* @keyframes ベース */
    @keyframes fade-in-slide-up {
      from { opacity: 0; translate: 0 1.25rem; }
    }
    [data-animate="fade-in-slide-up"] {
      animation: fade-in-slide-up 0.6s var(--ease-out-cubic) both;
    }

    /* stagger: 同じ @keyframes を遅延付きで子要素に適用 */
    [data-stagger="fade-in-slide-up"] > * {
      animation: fade-in-slide-up 0.6s var(--ease-out-cubic) both;
      animation-delay: var(--stagger-delay, 0s);
    }
  }
}
```

### 5.8 Utility

**責任**: 単一目的のスタイル上書き

**プレフィックス**: `u-`

**検証問い**: 「このスタイルは特定の Block に帰属せず、単一プロパティ（または密接に関連する最小プロパティセット）で完結するか？」 — Yes なら Utility。No なら Component / Project。

- MUST: `!important` を付与しなければならない
- SHOULD NOT: 特定の Block や Element に帰属できるスタイルを Utility に書くべきでない — Utility は特定の Block に帰属しない横断的かつ局所的な単一目的のスタイルに限る
- `!important` の使用制限については §4 を参照

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

MUST: Element（`__element`）は、対応する Block クラスが HTML 上に存在しなければならない。Block なしの Element は使用してはならない。CSS にルールセットがなくても、HTML 上に Block クラスが付与されていれば MUST 違反にはならない。BEM の「Element は常に Block の一部であり、Block から分離して使用してはならない」に基づく。
| Modifier | `.-{modifier}` | `.c-button.-primary` |
| State | `.is-{state}`, `.has-{state}` | `.is-active`, `.has-children` |

SHOULD: 層の識別のためにプレフィックス（§3 の層テーブルを参照）を使用すべきである。`@scope` 等の将来の CSS 機能によりプレフィックスが不要になる可能性があるため MUST としない。Tokens・Theme・Foundation はクラスセレクタを使用しないため、Animation は `data-animate` 属性セレクタを使用するため（§5.7 設計根拠を参照）、プレフィックスの対象外とする。

### Modifier と State の使い分け

- **Modifier（`.-xxx`）**: 静的なバリエーション。HTML に記述し、原則として変化しない
- **State（`.is-xxx`, `.has-xxx`）**: 動的な状態。JS やユーザー操作により変化する

SHOULD: Modifier は Component 層と Project 層で使用すべきである。SHOULD NOT: Layout 層では使用すべきでない（Layout は配置と空間のみを担当し、バリエーションは上位層で制御する）。SHOULD NOT: Animation 層（`data-animate` 属性セレクタ）と Utility 層は単一目的のスタイルであり、Modifier を使用すべきでない。

State のスタイルは、対象の Block が属する層に記述する（例: `.c-button.is-active` は Component 層、`[data-animate="fade-in"].is-active` は Animation 層）。

### Modifier に `-` を採用する理由

BEM の `--` ではなく `.-modifier` を採用する。

- HTML がコンパクトになる（`class="c-button -primary"` vs `class="c-button c-button--primary"`）
- CSS Nesting との相性が良い

### Element の深さ

SHOULD NOT: Element を 2 階層以上ネストすべきではない（`__element1__element2` は非推奨）。Element が深くなる場合はコンポーネントの分離を検討する。

### JS 連携

`js-` プレフィックスは設けない。JS からの要素取得には `data-*` 属性または ARIA 属性を使用する。

### ファイル名

SHOULD: ファイル名は主要クラス名と一致させるべきである — `{prefix}-{name}.css`。1 ファイルに関連クラスをグループ化する場合は、代表クラス名をファイル名とする（例: `u-hidden.css`）。Animation 層は `{種別}.css`（例: `fade-in.css`）とし、`data-animate` の値に対応させる（`animation/` ディレクトリが層を識別するため、ファイル名にプレフィックスは不要）。

例: `.c-button` → `c-button.css`, `.l-section` → `l-section.css`, `[data-animate="fade-in"]` → `animation/fade-in.css`

### カスタムプロパティ命名まとめ

| 層 | パターン | 例 |
|---|---|---|
| Tokens | `--{カテゴリ}-{名前}` | `--slate-600`, `--font-ja` |
| Theme | `--color-{役割}`, `--font-{役割}` | `--color-main`, `--font-body` |
| 公開 API | `--{対象}-{名前}` | `--section-padding-min`, `--badge-bg` |
| Private | `--_{名前}` | `--_font-size-min` |

---

## 7. Custom Properties

*This section is normative.*

### 3 層参照チェーン

カスタムプロパティは以下の 3 層を経由して使用する。

```
Tokens（値）→ Theme（意味）→ Foundation 以降（使用）
```

- SHOULD: Foundation 以降の層はブランドトークンについて Theme のセマンティック変数を参照すべきである
- SHOULD NOT: Foundation 以降の層がブランドトークンを直接参照すべきでない（Theme を経由する）
- MAY: グローバルトークン（ease, z-index 等）は Foundation 以降の層から直接参照してよい（Tokens 層のトークン分類を参照）

例外: Tokens の値を Theme 層でマッピングする際のみ、ブランドトークンへの直接参照が許される。

### 公開 API / プライベート命名規則

カスタムプロパティは公開 API とプライベートに分類する。

| 分類 | プレフィックス | 用途 | 例 |
|---|---|---|---|
| 公開 API | `--{対象}-{名前}` | 上位層または JS から上書きされる変数 | `--section-padding-min`, `--badge-bg`, `--stagger-delay` |
| プライベート | `--_` | Block 内部でのみ使用する変数 | `--_font-size-min`, `--_delay` |

SHOULD: 上位層（Project 等）から値を設定する変数、または JS から値を注入する変数は、`--{対象}-{名前}` の公開 API 命名を使用すべきである。外部からの契約として機能する変数にプライベート命名（`--_`）を使用すると、意図が不明確になる。

SHOULD: 公開 API のカスタムプロパティには `@property` で `inherits: false` を指定し、子孫への意図しない継承を防止すべきである。パフォーマンスの向上にも寄与する。

MAY: Layout 以降の層（Layout, Component, Project, Animation）でプライベートカスタムプロパティ（`--_xxx`）を定義してよい。プライベートカスタムプロパティは外部から参照・設定されることを想定しない内部実装である。

> **Example:**

```css
/* Layout 層で公開 API を定義 */
@layer layout {
  .l-section {
    --section-padding-min: 3.75rem;
    --section-padding-max: 6.25rem;

    padding-block: clamp(var(--section-padding-min), 8vi, var(--section-padding-max));
  }
}
```

上位層（Project 等）から公開 API の値を設定し、Layout や Component の振る舞いをページやセクションに合わせて変える。これは §5.6 の CSS 変数経由上書きパターンの基盤となる。

> **Example:**

```css
@layer project {
  /* Project の Block として公開 API を上書き */
  .p-about {
    --section-padding-min: 2.5rem;
    --section-padding-max: 3.75rem;
  }

  /* Project の Element として l-inner の振る舞いを配置先適応 */
  .p-about__inner {
    max-inline-size: 50rem;
  }
}
```

### インラインスタイル

MUST NOT: HTML マークアップに静的なインラインスタイルを記述してはならない。インラインスタイルはどの `@layer` にも属さず、mFLOCSS の「どこに何を書くか」を追跡可能にする設計思想と矛盾する。

MAY: JS から公開 API のカスタムプロパティの値を動的に注入してよい。CSS が「何をするか」を定義し、JS が「いつ・どの値で」を決める疎結合パターンとして許容する。

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

### 1 Block = 1 ファイル

SHOULD: 1 つの CSS ファイルには 1 つの Block を定義すべきである。複数の独立した Block を 1 ファイルに含めると、ファイル名と Block の対応が崩れる。

ただし Utility 層は例外とする。Utility は Block 構造を持たない単一目的のクラスであり、セマンティックな意味でグループ化することが推奨される（§5.8 参照）。

### layer-order.css

`@layer` の先制宣言のみを含む。スタイル定義は一切置かない。

### style.css（エントリポイント）

MUST: `layer-order.css` を最初に読み込み、その後層順にファイルをインポートしなければならない。

SHOULD: 各ファイルがどの層に属するかを明確にすべきである。方法はプロジェクトの規模やツールチェーンに応じて選択してよい。

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

SHOULD: コンポーネント単位のレスポンシブデザインには Container Queries を使用すべきである。ビューポート全体の変化には Media Queries、連続的な流体デザインには `clamp()` を使い分ける。

Container Queries の層責任（`container-type` / `container-name` / `@container` の配置先）は §5.4 Layout を参照。

### Container Queries の単位

SHOULD: Container Queries 内のサイズ指定には `cqi`（container query inline-size）を使用すべきである。`vw` はビューポート基準であり、コンテナ基準の設計と矛盾する。

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
| **ブランドトークン** | プロジェクトごとに変わるデザイン値（color, typography, structure）。Foundation 以降の層では Theme 経由で参照すべきである（初出: §5.1） |
| **グローバルトークン** | プロジェクト非依存の普遍値（ease, z-index, font-weight）。Foundation 以降の層から直接参照してよい（初出: §5.1） |
| **3 層参照チェーン** | Tokens（値）→ Theme（意味）→ Foundation 以降（使用）。カスタムプロパティの参照パスを規定する（初出: §7） |
| **公開 API（カスタムプロパティ）** | `--{対象}-{名前}` 形式の変数。上位層または JS から上書きされることを想定する外部インターフェース（初出: §7） |
| **プライベートカスタムプロパティ** | `--_` プレフィックスを持つ変数。Block 内部でのみ使用し、外部からの参照・設定を想定しない（初出: §7） |
| **先制宣言** | `layer-order.css` における `@layer` の優先順位宣言。全スタイル定義に先行して記述される（初出: §4） |
| **装飾的アニメーション** | 視覚演出としての動き。無効化しても機能に影響しない。Animation 層に分離し、2 ガード原則を適用する（初出: §5.7） |
| **機能的トランジション** | インタラクションフィードバックとしての動き。ユーザー操作に対する応答であり、対象の Block が属する層（Component または Project）に記述する（初出: §5.7） |
| **2 ガード原則** | Animation 層で `prefers-reduced-motion` と `scripting` の 2 条件を考慮すること。推奨は `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガード（初出: §5.7） |
| **Block** | BEM における独立した意味のあるエンティティ。プレフィックス付きクラス名（`.c-card`, `.p-hero` 等）で表現する（初出: §6） |
| **Element（`__element`）** | Block の一部。命名は `.__{element}` の形式。Block なしでの使用禁止等の規範的定義は §6 を参照（初出: §6） |
| **Modifier（`.-xxx`）** | 静的なバリエーション。Component 層と Project 層で使用する（SHOULD）。HTML に記述し、原則として変化しない（初出: §6） |

---

## Appendix B: References

### Normative References

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
- **[CSS-CASCADE-5]** Atkins Jr., T.; Rivoal, F.; Lilley, C., "CSS Cascading and Inheritance Level 5", W3C Candidate Recommendation.
- **[CSS-PROPERTIES-VALUES-1]** Atkins-Bittner, T.; Stearns, A.; Whitworth, G., "CSS Properties and Values API Level 1", W3C Working Draft.

### Informative References

- **[FLOCSS]** Hiloki, "FLOCSS — Foundation Layout Object CSS". https://github.com/hiloki/flocss
- **[BOOK]** shunei,『そのFLOCSS、なぜそこに書いた？』. https://zenn.dev/shunei/books/mflocss-design

---

## Appendix C: Changes

変更履歴は [CHANGELOG.md](./CHANGELOG.md) を参照。
