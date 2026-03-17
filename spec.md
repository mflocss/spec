# mFLOCSS 仕様 v1.1

> **ステータス**: ドラフト
> **最終更新**: 2026-03-11
> **著者**: shunei

---

## 1. Introduction

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークである。

「どの層に、なぜ書くか」という問いに対し、明確な判断基準を提供する。`@layer` ベースの 8 層フラットアーキテクチャを採用する。

### 対象範囲

- フレームワーク非依存（素の CSS を対象とする）
- プリプロセッサを前提としない（Sass 等との併用は妨げない）
- JavaScript フレームワーク固有の CSS-in-JS は対象外

### 不変原則

mFLOCSS は以下の 4 原則に基づく。層数が変わってもこれらは変わらない。

1. **関心の分離** — 異なる問いに答えるスタイルは異なる層に分離する
2. **@layer による構造的制御** — 詳細度の問題を命名規則ではなくブラウザの仕組みで解決する
3. **判断基準の明示** — 各層に「何を書くか」だけでなく「なぜその層か」の基準がある
4. **CSS の進化への追従** — 層構成は CSS の進化に応じて適応させる設計余地を持つ（具体的な検討事項は README を参照）

---

## 2. Conformance

### 要求レベル

本仕様のキーワード MUST / MUST NOT / SHOULD / SHOULD NOT / MAY は RFC 2119（技術仕様における要求レベルの定義標準）に基づく。

| キーワード | 意味 |
|---|---|
| MUST | 絶対的な要求。違反は非準拠となる |
| MUST NOT | 絶対的な禁止。違反は非準拠となる |
| SHOULD | 推奨。正当な理由がある場合に限り逸脱を許容する |
| SHOULD NOT | 非推奨。正当な理由がある場合に限り逸脱を許容する |
| MAY | 任意。プロジェクトの判断に委ねる |

### 準拠条件

mFLOCSS v1.1 に準拠するとは、本仕様の全 MUST / MUST NOT ルールに違反しないことを意味する。

### バージョニング

- 層の追加・削除・統合はメジャーバージョン変更とする
- 既存層内のルール追加・変更はマイナーバージョン変更とする

---

## 3. Layer Architecture

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

### Portability Test

Component 層と Project 層の境界を判定する基準テスト。

> 「そのパーツを別のサイトにそのまま持っていけるか？」

- Yes → Component
- No → Project

---

## 4. Layer Order Declaration

### 先制宣言

MUST: `@layer` の優先順位宣言をエントリポイント CSS の先頭で行わなければならない。

```css
/* layer-order.css */
@layer tokens, theme, foundation, layout, component, project, animation, utility;
```

この宣言が全ての `@import` に先行しなければならない。

### @layer と @property

MUST: `@property` を使用する場合は `@layer` の外に配置しなければならない。CSS 仕様上、`@layer` 内の `@property` は無視される。`@property` 自体の使用は任意である。

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

- MUST: 要素セレクタのみを使用しなければならない（クラスセレクタ禁止）
- MUST: `:where()` で詳細度をゼロに保たなければならない
- MUST: ブランドトークンについては Theme のセマンティック変数を参照しなければならない（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）

SHOULD: reset / base / form の 3 ファイルに分割する。form を独立させる理由は、フォーム要素（input, select, textarea, button）はブラウザ間のデフォルトスタイル差異が最も大きく、正規化のコード量が多くなるためである。

> **注記**: リセット CSS はアタッチメント方式とする。リファレンス実装が参考を提供するが、各プロジェクトに適したリセットを選定・適用すること。リセット CSS の内部実装は本仕様の準拠対象外とする。

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

SHOULD: `container-type: inline-size` を宣言し、Container Queries の基盤を提供する。

プライベートカスタムプロパティ（`--_xxx`）の使用については §7 Custom Properties を参照。

**検証問い（Layout Test）**: 「このスタイルを変えると、中身の見た目（色・文字・装飾）が変わるか？」 — Yes なら Layout ではない。

```css
@layer layout {
  .l-section {
    --_padding-min: 3.75rem;
    --_padding-max: 6.25rem;

    container-type: inline-size;
    padding-block: clamp(var(--_padding-min), 8vi, var(--_padding-max));
  }
}
```

### 5.5 Component

**責任**: 配置先に左右されない再利用可能なパーツ

**プレフィックス**: `c-`

- MUST: Portability Test に合格しなければならない（「別サイトにそのまま持っていけるか？」）
- MUST: 特定のサイトでしか使えない見た目にしてはならない
- MUST: ブランドトークンについては Theme のセマンティック変数のみを参照しなければならない（グローバルトークンの直接参照は許容 — Tokens 層のトークン分類を参照）
- MUST NOT: 外部レイアウトに影響するプロパティ（ルート要素の `margin`, `position: fixed/sticky`, ルート要素の `overflow` 等）を Component のルート要素に含めてはならない — 配置は使う側の責任（Responsibility Test）である。ただし Component 内部の Element 間の余白や内部配置（`position: relative` / `absolute`）は許容する

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
| Project → Component | コンポーネントの配置先適応 | `.p-hero > .c-button { ... }` |
| Modifier（Component 自身） | 汎用バリエーション | `.c-button.-primary` |

Modifier は Component に内包される再利用可能なバリエーション。Project 上書きはサイト固有のデザイン要件に基づき Component のスタイルを調整するもの。

Layout の振る舞いをページやセクションに合わせて変えたい場合は、Project の Block または Element として定義する（例: `class="l-section p-about"`, `class="l-inner p-about__inner"`）。

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

- MUST: `prefers-reduced-motion` と `scripting` の 2 条件を考慮しなければならない（モーション軽減時にアニメーションが無効になり、JS 無効時に要素が不可視にならないことを保証する）

この 2 条件の考慮を **2 ガード原則** と呼ぶ。

SHOULD: `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガードを使用する。条件を満たさない場合にブロック全体が不適用になり、フォールバック安全性が高い。

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
- MUST NOT: 層の設計で解決できるスタイルを Utility に書いてはならない — Utility の増殖は設計の不在を示す
- MUST NOT: Utility 層以外の全層で `!important` を使用してはならない

**適切な用途**: アクセシビリティ非表示（`u-visually-hidden`）、レスポンシブ表示制御

**不適切な用途**: 色の定義、レイアウトの組み立て、コンポーネントの装飾

**ファイル構成**: SHOULD: セマンティックな意味でグループ化する。1 クラス 1 ファイルではなく、関連するユーティリティをまとめる（例: `u-hidden.css` に `u-visually-hidden` と `u-hidden-sp` をまとめる）。Utility は最小限に留まるため、ファイル数も最小で済む。

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

MAY: Layout 以降の層（Layout, Component, Project, Animation）でプライベートカスタムプロパティ（`--_xxx`）を定義してよい。`--_` プレフィックスは内部変数であることを命名で明示する。外部 API ではないことを示す。

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

### アプローチの選択

SP ファースト / PC ファーストはプロジェクトごとに判断する。本仕様はどちらも許容する。

ブレークポイントもプロジェクトごとに設定する。768px / 1024px は参考値であり、本仕様の規定値ではない。

### 3 つの手法と使い分け

| 手法 | 用途 | 適用例 |
|---|---|---|
| Container Queries | コンポーネント単位のレスポンシブ（主要手段） | カードの画像/コンテンツの縦横切替、テーブルのレイアウト変更 |
| Media Queries | ビューポート全体の離散的変化 | ナビゲーション切替、カラム数変更 |
| `clamp()` | 連続的な流体デザイン | フォントサイズ、余白の滑らかな変化 |

### Container Queries の層責任

| 責任 | 層 |
|---|---|
| `container-type` の宣言 | Layout |
| `@container` によるスタイル切替 | Component / Project |

SHOULD: Layout 層で `container-type: inline-size` を宣言し、Component / Project 層が `@container` で参照する構成とする。

### Container Queries の単位

SHOULD: Container Queries 内のサイズ指定には `cqi`（container query inline-size）を使用する。`vw` はビューポート基準であり、コンテナ基準の設計と矛盾する。

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

## 10. Appendix A: Layer Judgment Flowchart

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

Step 5: prefers-reduced-motion で無効化すべき動きか？
  └─ Yes → Animation

Step 6: 局所的な単一目的の微調整か？
  └─ Yes → Utility
```

### よくある誤りパターン

**A. 機能近接バイアス**: 「テーブルのスクロールラッパー → Component」と判断する誤り。スクロール制御は使う側のコンテナの制約であり、テーブルパーツ自体の責任ではない。正しくは Project の Element。

**B. Layout への過干渉**: `l-section` に `text-align: center` を追加する誤り。テキスト整列は視覚的プロパティであり、Layout の責任（配置と空間）を超えている。正しくは Project。

**C. ラッパーの帰属誤り**: `overflow-x: auto` を Component のラッパーとして定義する誤り。コンテナ幅の制約対応はサイト固有のデザイン要件に依存する。正しくは `p-xxx__table-wrap` として Project の Element に持たせる。

---

## 11. Appendix B: Glossary

| 用語 | 定義 |
|---|---|
| **Portability Test** | 「そのパーツを別のサイトにそのまま持っていけるか？」— Component と Project の境界を判定する基準テスト |
| **Layout Test** | 「このスタイルを変えると、中身の見た目（色・文字・装飾）が変わるか？」— Layout 層の責任範囲を判定する検証問い |
| **Responsibility Test** | 「誰の責任か？」— パーツ自身の責任（Component）か、使う側のデザイン要件（Project）かを判定する問い |
| **ブランドトークン** | プロジェクトごとに変わるデザイン値（color, typography, structure）。Foundation 以降の層では Theme 経由で参照しなければならない |
| **グローバルトークン** | プロジェクト非依存の普遍値（ease, z-index, font-weight）。Foundation 以降の層から直接参照してよい |
| **3 層参照チェーン** | Tokens（値）→ Theme（意味）→ Foundation 以降（使用）。カスタムプロパティの参照パスを規定する |
| **プライベートカスタムプロパティ** | `--_` プレフィックスを持つ変数。内部 API であり、外部からの直接依存を想定しない |
| **先制宣言** | `layer-order.css` における `@layer` の優先順位宣言。全スタイル定義に先行して記述される |
| **2 ガード原則** | Animation 層で `prefers-reduced-motion` と `scripting` の 2 条件を考慮すること。推奨は `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガード |
