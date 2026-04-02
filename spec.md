# mFLOCSS 仕様

---

## Abstract

*This section is informative.*

mFLOCSS は、CSS の記述を 8 つの層に分類する思考フレームワークである。「このスタイルをどの層に書くか」という判断を体系化し、設計の一貫性と長期的な保守性を実現する。本仕様はその判断基準・命名規則・ファイル構成を厳密に定義する。

---

## Status of This Document

*This section is informative.*

本文書は mFLOCSS 仕様書のドラフト版である。v1.0 で正式版となる予定であり、v1.0 リリース以前は仕様の内容が変更される可能性がある。

**最終更新**: 2026-03-28  
**著者**: shunei

---

## 1. Overview

### 1.1 Introduction

*This section is informative.*

mFLOCSS は、CSS の設計判断を体系化する思考フレームワークであり、ルールブックではない。「どの層に、なぜ書くか」という問いに対し、明確な判断基準を提供する。`@layer` ベースの 8 層フラットアーキテクチャを採用する。

本仕様はその判断基準を厳密に定義する。

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

1. **一方向依存** — 上位層は下位層を参照してよいが、逆方向は禁止する（詳細は §3）

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

> **注記（Informative）**: SHOULD からの逸脱は、不変原則（§1.2）を理解した上で判断すること。逸脱の理由をコード内コメントまたは設計文書に残すことを推奨する。

### 準拠条件

mFLOCSS に準拠するとは、本仕様の全 MUST / MUST NOT ルールに違反しないことを意味する。

MUST は `@layer` の構造的整合性・アクセシビリティ・命名体系の維持に限定される。

設計判断の品質（層の選択・トークン参照チェーン等）は SHOULD で推奨し、遵守するほど設計の一貫性と保守性が向上する。

### バージョニング

- 層の追加・削除・統合はメジャーバージョン変更とする
- 既存層内のルール追加・変更はマイナーバージョン変更とする

---

## 3. Layer Architecture

*This section is normative.*

mFLOCSS は 8 つの層で構成される。本章以降で使用する Block・Element 等の命名規則は §6 で定義する。

| 順序 | 層名 | プレフィックス | 責任 |
|---|---|---|---|
| 1 | token | — | デザイントークンの定義（プリミティブ値とセマンティック変数） |
| 2 | reset | — | ブラウザデフォルトの正規化 |
| 3 | foundation | — | 要素の基本スタイル |
| 4 | layout | `l-` | 位置と空間の配置 |
| 5 | component | `c-` | 配置先に左右されない再利用可能なパーツ |
| 6 | project | `p-` | サイト固有のパーツとデザイン要件 |
| 7 | animation | — | 動きの分離管理 |
| 8 | utility | `u-` | 単一目的のスタイル上書き |

### 層間の依存方向

- MAY [下位参照の許可]: 上位層（番号が大きい層）は下位層を参照してよい
- MUST NOT [逆方向参照の禁止]: 下位層から上位層のクラスやカスタムプロパティを参照してはならない（例: Component 層が Project 層のクラスに依存してはならない）

依存方向ルールは **CSS の参照方向** に適用される。HTML の入れ子構造はこのルールの対象外である。Component の中に Project を配置し、その中に Component を置く構造（例: `.c-modal` > `.p-login-form` > `.c-button`）は、CSS で他の層のクラスを参照しない限り違反にならない。

### Layer Judgment Flowchart

> **注記（Informative）**: このフローチャートは層判断の参考ガイドであり、規範的ルールではない。

スタイルをどの層に書くべきかを 6 ステップで判断する。

```
Step 1: デザイントークン（色の値、フォント名、z-index 値等）か？
  └─ Yes → Token（プリミティブ値とセマンティック変数を同層で管理）

Step 2: ブラウザデフォルトの正規化か？要素の基本スタイルか？
  ├─ 正規化（リセット CSS）→ Reset
  └─ 要素の基本スタイル → Foundation

Step 3: 配置と空間だけか？（色・文字・装飾に触れないか？）
  └─ Yes → Layout

Step 4: Portability Test — 別のサイトにそのまま持っていけるか？
  ├─ Yes → Component
  └─ No → Project
  ※ 判断が曖昧な場合は、§5.5 の補助テスト（Responsibility Test）を使用する。

※ Step 5-6 は Step 1-4 の判定を覆さない追加チェックである。
  Step 1-4 で判定した層のスタイルの中に、Animation・Utility の条件を
  満たす部分があれば、その部分を該当層に分離する。
  （例: Step 4 で Component と判定したスタイルに装飾的アニメーションが
  含まれる場合、その動きの部分だけを Animation 層に切り出す）

Step 5: 装飾的アニメーション（視覚演出）か？
  ├─ Yes → Animation（2 ガード原則を適用）
  └─ 機能的トランジション（インタラクションフィードバック）→ Component / Project に残す

Step 6: 局所的な単一目的の微調整か？
  └─ Yes → Utility
```

#### よくある誤りパターン

**A. 機能近接バイアス（思い込み）**: 「テーブルのスクロールラッパー → Component」と判断する誤り。`overflow-x: auto` 等のスクロール制御は使う側のコンテナの制約であり、テーブルパーツ自体の責任ではない。正しくは `p-xxx__table-wrap` として Project の Element に持たせる。

**B. Layout への過干渉**: `l-section` に `text-align: center` を追加する誤り。テキスト整列は視覚的プロパティであり、Layout の責任（配置と空間）を超えている。正しくは Project。

---

## 4. Layer Order Declaration

*This section is normative.*

CSS `@layer` の構造的整合性を維持するために必要な宣言ルールを定義する。先制宣言・`@property` の配置・`!important` の使用制限・外部 CSS の取り込みを規定する。

**要求レベル:**

- MUST [先制宣言の実施]: CSS Cascading and Inheritance Level 5 [CSS-CASCADE-5] に定義される `@layer` による層間の優先順位宣言を起点ファイル（エントリポイント）CSS の先頭で、全ての `@import` に先行して行わなければならない。
- MUST [@property のレイヤー外配置]: `@property` を使用する場合は `@layer` の外に配置しなければならない。現行の CSS 仕様 [CSS-PROPERTIES-VALUES-1] において、`@layer` 内の `@property` は無視される。`@property` 自体の使用は任意である。
- MUST NOT [!important の使用制限]: Reset 層および Utility 層を除く全層で `!important` を使用してはならない。Reset 層は外部リセット CSS を取り込むため、その内部実装における `!important` の有無は本仕様の準拠対象外とする（§5.2 注記参照）。この制約により優先度逆転の複雑性を回避する。
- MUST [外部 CSS の層取り込み]: 外部 CSS は `@import url() layer()` または npm + バンドラーを使用し、いずれかの層に取り込まなければならない。

### 先制宣言

> **注記（Informative）**: 先制宣言は `layer-order.css` に配置し、スタイル定義は一切置かない（§8 参照）。

> **Example（MUST [先制宣言の実施]）:**

```css
/* layer-order.css */
@layer token, reset, foundation, layout, component, project, animation, utility;
```

### @layer と @property

> **注記（Informative）**: `@property` はカスタムプロパティの型・初期値・継承を明示的に定義するための CSS 機能。`@layer` の外（`property.css`）に配置する（§8 参照）。

### !important の優先度逆転

> **注記（Informative）**: `@layer` 内で `!important` を使用した場合、通常とは逆順で優先される（先に宣言された層が勝つ）。MUST NOT [!important の使用制限] によりこの優先度逆転の複雑性を回避する。

### 外部 CSS の層配置

> **注記（Informative）**: `@layer` に属さない CSS（unlayered CSS）は全ての layered CSS より優先される。外部ライブラリの CSS を `<link>` タグ等で直接読み込むと、mFLOCSS の層構造を貫通し、意図しない上書きが発生する。外部 CSS の層配置は自作 CSS と同じ判断基準で決定する。出自（外部 / 自作）による特別扱いは行わない。

| 外部 CSS の種類 | 配置先 |
|---|---|
| リセット系ライブラリ | `layer(reset)` |
| UI コンポーネント系ライブラリ | `layer(component)` |

---

## 5. Layer Definitions

*This section is normative.*

### 5.1 Token

**責任**: デザイントークンの定義（プリミティブ値とセマンティック変数）

**Token 層の責任:**

1. **デザイントークンの定義**: プリミティブ値（生の値）とセマンティック変数（意味を持つマッピング）の管理
2. **ダークモード / テーマ切替の完結**: `color-scheme` とセマンティック変数でテーマを Token 層内に閉じ込める
3. **ブランドトークンとグローバルトークンの分離**: プロジェクト固有の値と汎用的な値を区別する

**検証問い（Token Test）**: 「この値はデザイントークン（色の値、フォント名、余白、z-index 等）か？または単位変換・計算のためのユーティリティ変数（`--px` 等）か？」

- Yes → Token
- No → 他の層

**要求レベル:**

- MUST [:root セレクタ限定]: `:root` セレクタのみを使用しなければならない
- MUST NOT [他層カスタムプロパティ参照禁止]: 他の層のカスタムプロパティを参照してはならない
- SHOULD [プリミティブとセマンティックの分離]: コンテキスト依存の値を持つカテゴリは、プリミティブ変数とセマンティック変数を同一ファイル内で分離すべきである（トークンの分類を参照）
- SHOULD [テーマ切替の Token 完結]: ダークモード / テーマ切替は Token 層のセマンティック変数で完結させるべきである
- MAY [カテゴリ別ファイル分割]: カテゴリ別にファイルを分割してよい（color / typography / space / structure / ease / z-index 等）

**トークンの分類**:

Token 層はプリミティブ変数とセマンティック変数を同一層内で管理する。コンテキスト依存でないカテゴリは値のみを定義する。

| 分類 | 性質 | 例 |
|------|------|-----|
| プリミティブ変数 | 生の値の定義 | `--_slate-600`, `--_slate-900` |
| セマンティック変数 | 意味を持つマッピング | `--color-main`, `--color-surface`, `--font-size-body` |
| グローバルトークン | 多くのプロジェクトで共通して使える値 | `--ease-out-cubic`, `--z-header` |
| 計算ヘルパー | 単位変換・計算のためのユーティリティ変数 | `--px: calc(1rem / 16)`（`calc(数値 * var(--px))` で rem に変換） |

判断基準: 「ブランドやプロジェクトが変わったとき、この値を変更する必要があるか？」 — Yes ならブランドトークン（プリミティブ + セマンティック）、No ならグローバルトークン。計算ヘルパーはブランドに依存しないユーティリティ変数として Token 層に配置する。

命名規則は §6 カスタムプロパティ命名まとめを参照。

> **Example（MUST [:root セレクタ限定]、SHOULD [プリミティブとセマンティックの分離]）:**

```css
@layer token {
  :root {
    /* プリミティブ（color） */
    --_slate-900: #1a202c;
    --_slate-100: #f1f5f9;

    /* セマンティック（color） */
    color-scheme: light dark;
    --color-main: light-dark(var(--_slate-900), var(--_slate-100));

    /* グローバルトークン */
    --font-family: "Noto Sans JP", sans-serif;
    --ease-out-cubic: cubic-bezier(0.33, 1, 0.68, 1);
  }
}
```

### 5.2 Reset

**責任**: ブラウザデフォルトの正規化

**Reset 層の責任:**

1. **ブラウザデフォルトの初期化**: リセットまたはノーマライズによるブラウザ間差異の吸収
2. **リセット CSS の隔離**: 独立層として隔離し、他層との詳細度競合を防止

**検証問い（Reset Test）**: 「このスタイルはブラウザデフォルトの初期化か？」

- Yes → Reset
- No → Foundation 以降の層

**要求レベル:**

- MAY [Reset 層の使用任意]: Reset 層の使用は任意である
- MUST [Reset 層の責任限定]: Reset 層を使用する場合、ブラウザデフォルトの初期化に限定しなければならない（自作・外部を問わない）。プロジェクト固有のスタイルを Reset 層に記述してはならない
- SHOULD [適切なリセット CSS の配置]: Reset 層を使用する場合、プロジェクトに適したリセット CSS を選定し、この層に配置すべきである

> **注記（Informative）**: リセット CSS は自作でも外部ライブラリでもよい。リファレンス実装が参考を提供するが、各プロジェクトに適したリセットを選定・適用すること。リセット CSS の内部実装は本仕様の準拠対象外とする。`:where()` を使用していない外部リセット CSS でも、独立層として隔離されるため、Foundation 以降の層との詳細度競合を防止できる。

> **Example（SHOULD [適切なリセット CSS の配置]）:**

```css
/* reset/reset.css */
@import url("modern-normalize.css") layer(reset);
```

### 5.3 Foundation

**責任**: 要素の基本スタイル

**Foundation 層の責任:**

1. **要素の基本スタイル**: 全ページ共通のタイポグラフィ、行間、色の初期設定
2. **フォーム要素の正規化**: ブラウザ間の差異を吸収する基本スタイル
3. **詳細度ゼロの維持**: `:where()` で詳細度を抑え、上位層からの上書きを妨げない

**検証問い（Foundation Test）**: 「このスタイルは、要素の基本スタイルの定義か？」

- Yes → Foundation
- No（特定のコンテキストに依存する）→ 上位層

**要求レベル:**

- MUST NOT [クラスセレクタ・ID セレクタの使用禁止]: クラスセレクタ・ID セレクタを使用してはならない。Foundation 層は要素型セレクタのみを使用し、全ページ共通の基本スタイルを定義する
- SHOULD [属性セレクタの :where() 内使用]: 属性セレクタ、擬似クラスは、`:where()` 内で要素型セレクタと組み合わせて使用すべきである。擬似要素は `:where()` の引数に使用できないため、外に記述する（例: `:where(p)::before`）
- SHOULD [:where() による詳細度ゼロ]: `:where()` で詳細度をゼロに保つべきである。`@layer` による層順序制御が詳細度の予測可能性を構造的に担保するため、`:where()` は推奨とする
- SHOULD [トークン参照の規則遵守]: トークン参照は §7 に従う
- SHOULD [base と form の分離]: base と form を分離すべきである。form を独立させる理由は、フォーム要素（input, select, textarea, button）はブラウザ間のデフォルトスタイル差異が大きく、正規化のコード量が多くなるためである。

> **Example（MUST NOT [クラスセレクタ・ID セレクタの使用禁止]、SHOULD [:where() による詳細度ゼロ]、SHOULD [base と form の分離]）:**

```css
/* foundation/base.css — 自作ベーススタイル */
@layer foundation {
  :where(body) {
    font-family: var(--font-family);
    color: var(--color-main);
    background-color: var(--color-surface);
    line-height: 1.5;
  }
}

/* foundation/form.css — フォーム要素の基本スタイル */
@layer foundation {
  :where(input, select, textarea, button) {
    font: inherit;
  }
}
```

### 5.4 Layout

**責任**: 位置と空間の配置

**プレフィックス**: `l-`

**Layout 層の責任:**

1. **ページの骨格（ストラクチャ）**: ヘッダー、メインコンテンツ、セクション、フッターなど、ページ全体の構造を定義する
2. **空間の確保**: セクション間の余白（`padding-block`）、コンテンツ幅の制約（`max-inline-size`）
3. **配置制御**: `position: sticky` / `z-index` などの構造的な配置
4. **Container Queries の基盤**: `container-type` / `container-name` の宣言
5. **公開 API の提供**: カスタムプロパティで上位層（Project）に値の設定を委ねる

**検証問い（Layout Test）**:

- 「このスタイルは、要素の配置・寸法・空間の確保を担っているか？」 — Yes なら Layout
- 「中身の見た目（色・文字・装飾・影・透明度）が変わるか？」 — Yes なら Layout ではない

**要求レベル:**

- SHOULD NOT [視覚プロパティの排除]: 見た目に関するプロパティ（`color`, `font-size`, `background-color`, `border`, `text-align` 等の視覚的プロパティ）を宣言すべきでない
- MAY [Container Queries 基盤の宣言]: Container Queries を使用する場合、`container-type: inline-size` を宣言し、Container Queries の基盤を提供してよい
- MAY [名前付きコンテナの定義]: `container-name` を併用し、名前付きコンテナを定義してよい。セクション単位で名前付きコンテナを定義すると、`@container` での参照先を明示できる

#### Container Queries の層責任

> **注記（Informative）**: Container Queries は複数層にまたがる仕組みであるため、層責任の全体像をここにまとめて示す。各層の個別ルールは該当セクション（§5.5, §5.6）に従う。

| 責任 | 層 |
|---|---|
| `container-type` の宣言 | Layout |
| `container-name` の定義 | Layout（MAY） |
| `@container` によるスタイル切替 | Component / Project |

プライベートカスタムプロパティ（`--_xxx`）の使用については §7 Custom Properties を参照。

> **Example（MAY [Container Queries 基盤の宣言]、MAY [名前付きコンテナの定義]）:**

```css
@layer layout {
  /* 配置・重なり・コンテナ定義 */
  .l-header {
    position: sticky;
    inset-block-start: 0;
    z-index: var(--z-header);
    container-type: inline-size;
  }

  /* 公開 API: カスタムプロパティで上位層に値の設定を委ねる */
  .l-section {
    --section-padding: 3.75rem;

    container-type: inline-size;
    padding-block: var(--section-padding);
  }
}
```

### 5.5 Component

**責任**: 配置先に左右されない再利用可能なパーツ

**プレフィックス**: `c-`

**Component 層の責任:**

1. **再利用可能なパーツの定義**: 配置先に依存しない、自己完結した UI パーツ
2. **Portability Test**: 「別のサイトにそのまま持っていけるか？」で Component か Project かを判定
3. **公開 API の提供**: カスタムプロパティで上位層に値の設定を委ねる

**検証問い（Portability Test）**: 「そのパーツを別のサイトにそのまま持っていけるか？」

- Yes → Component
- No → Project（§5.6）

**補助テスト（Responsibility Test）**: Portability Test で判断が曖昧な場合にのみ使用する。Portability Test が明確に Yes または No を返す場合、Responsibility Test の結果に関わらずその判定に従う。

> 「このスタイルは、パーツ自身の視覚的責任か？」

- Yes → Component: ボタンの背景色、カードの角丸
- No（使う側のデザイン要件）→ Project: ヒーロー内のボタンのサイズ変更、カードの配置間隔

**要求レベル:**

- SHOULD [Portability Test の合格]: Portability Test に合格すべきである
- SHOULD NOT [サイト固有スタイルの排除]: 特定のサイトでしか使えない見た目にすべきでない
- SHOULD [迷ったら Component 優先]: Component と Project の判断に迷った場合は Component とすべきである。Project で上書き可能だが、逆（Project → Component への汎化）は困難であるため。Portability Test で明確に No と判断できる場合はこの限りではない
- SHOULD [トークン参照の規則遵守]: トークン参照は §7 に従う
- SHOULD NOT [外部レイアウト影響プロパティの排除]: 外部レイアウトに影響するプロパティ（ルート要素の `margin`, `position: fixed/sticky`, ルート要素の `overflow` 等）を Component のルート要素に含めるべきでない — 配置は使う側の責任（Responsibility Test）である
- MAY [他層要素の内包]: HTML 上で任意の層の要素を内包してよい（§3 依存方向を参照）。内包しても Portability Test に合格するなら Component のままである

> **注記（Informative）**: Component 内部の Element（`__element`）間の余白（`margin`, `gap`）や内部配置（`position: relative` / `absolute`）は上記 SHOULD NOT の対象外である。これらは Component 自身の視覚的責任に該当する

> **Example（SHOULD [Portability Test の合格]、SHOULD [トークン参照の規則遵守]）:**

```css
@layer component {
  .c-card {
    /* セマンティック変数を参照 */
    padding: var(--space-lg);
    background-color: var(--color-surface);

    /* プライベートカスタムプロパティ: Block 内部の計算用 */
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

**責任**: サイト固有のパーツとデザイン要件

**プレフィックス**: `p-`

**Project 層の責任:**

1. **サイト固有のパーツとデザイン要件**: ページ / セクション単位の固有スタイル
2. **Component / Layout の上書き**: CSS 変数経由 / 直接プロパティ / Modifier の 3 パターン
3. **セクションルートの管理**: ページ内の各セクションに Project Block を付与

**検証問い**: Portability Test（§5.5）で No → Project

**要求レベル:**

- SHOULD NOT [Component 相当スタイルの Project 記述禁止]: Portability Test（§5.5）に合格するスタイルを Project 層に記述すべきでない。該当するスタイルは Component 層（§5.5）に記述する
- SHOULD [セクションルートへの Project Block 付与]: サイト固有のデザイン要件があるセクションには、セクションルートに Project Block を付与し、セクション内の要素は Element として構築すべきである。インラインスタイルについては §7 Custom Properties を参照
- SHOULD [CSS 変数経由上書きの優先]: 直接プロパティ上書きと CSS 変数経由のどちらも使用できる場合は、CSS 変数経由を優先すべきである（上書きパターンを参照）
- SHOULD [ページ・機能単位のファイル分割]: ページ単位または機能単位でファイルを分割すべきである
- MAY [他層要素の内包]: HTML 上で任意の層の要素を内包してよい（§3 依存方向を参照）
- MAY [Layout と Component のみの構成]: サイト固有のスタイリングが不要なセクションは Layout と Component のみで構成してよい
- MAY [拡張用 Element クラスの先行付与]: Project が内包する Component や Layout の要素に、現時点で固有スタイルがなくても Project の Element クラスを付与してよい（例: `class="c-section-heading p-about__heading"`）。拡張点として機能する

#### 上書きパターン

Project 層は上位層として、Component のスタイルをページやセクションに合わせて上書きできる。

| パターン | 用途 | 例 |
|---|---|---|
| 直接プロパティ上書き | 特定の配置先でパーツのスタイルを変更 | `.p-hero .c-button { font-size: ... }` |
| CSS 変数経由 | Component/Layout が公開する公開 API カスタムプロパティの値を設定 | `.p-about { --section-padding: 2.5rem; }` |
| Modifier（Component 層 / Layout 層 / Project 層で定義） | 汎用バリエーション | `.c-button.-primary` |

CSS 変数経由は Component/Layout の内部構造に依存しないため保守性が高い。

Modifier は Component に内包される再利用可能なバリエーション。Project 上書きはサイト固有のデザイン要件に基づき Component のスタイルを調整するもの。

**境界事例**: 特定ページでしか使わないボタンサイズの変更は Modifier か Project 上書きか？ — 他のページでも再利用しうるなら Modifier（`.-large`）、そのページ固有なら Project 上書き（`.p-hero > .c-button { font-size: ... }`）。判断基準は Portability Test と同じ「他でも使うか？」。

Layout や Component の振る舞いをページやセクションに合わせて変えたい場合は、Project の Block または Element として定義する（例: `class="l-section p-about"`, `class="c-button p-about__cta"`）。

> **Example（SHOULD [CSS 変数経由上書きの優先]、SHOULD [セクションルートへの Project Block 付与]）:**

```css
@layer project {
  /* Block: Layout の公開 API を上書き */
  .p-about {
    --section-padding: 2.5rem;
  }

  /* Element: Layout の配置先適応 */
  .p-about__inner {
    max-inline-size: 50rem;
  }

  /* Project 固有のパーツ */
  .p-hero__lead {
    text-align: center;
  }
}
```

### 5.7 Animation

**責任**: 動きの分離管理

**Animation 層の責任:**

1. **装飾的アニメーションの分離管理**: Component / Project から動きを分離し一元管理
2. **アクセシビリティガードの保証**: 2 ガード原則（prefers-reduced-motion + scripting）
3. **状態遷移の制御**: `data-animate` + `data-*` 属性による表示制御

**検証問い（Animation Test）**: 「その動きを無効化しても、インタラクションの意味が伝わるか？」

- Yes（なくても伝わる）→ 装飾的 → Animation
- No（ないと操作感が損なわれる）→ 機能的 → Component/Project

**要求レベル:**

- MUST [2 ガード原則の実装]: Animation 層のスタイルは、`prefers-reduced-motion: reduce` 環境でアニメーションが無効になり、`scripting: none` 環境で要素が不可視にならない実装としなければならない
- SHOULD [装飾的アニメーションの Animation 層分離]: 装飾的アニメーションは Animation 層に分離し、2 ガード原則を適用すべきである
- SHOULD [統合ガードの使用]: `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガードを使用すべきである。条件を満たさない場合にブロック全体が不適用になり、フォールバック（代替値）安全性が高い
- SHOULD [transform を含む機能的トランジションのガード]: 機能的トランジションのうち `transform`（`translate` / `rotate` / `scale` など）を含むものは、`prefers-reduced-motion: no-preference` のガードを適用すべきである。前庭障害のトリガーになりうるためである。色変化（`color` / `background-color` 等）や `opacity` のみのトランジションはガード不要である
- SHOULD [@keyframes 名と data-* 属性値の一致]: `@keyframes` 名は対応する `data-*` 属性の値と一致させるべきである（@keyframes 命名を参照）
- MAY [機能的トランジションの所属層への記述]: 機能的トランジションは、対象の Block が属する層（Component または Project）に記述してよい

この `prefers-reduced-motion` と `scripting` の 2 条件の考慮を **2 ガード原則** と呼ぶ。統合ガードを使用しない場合は、`prefers-reduced-motion: reduce` でアニメーション関連プロパティが初期値に解決されること、`scripting: none` で要素の可視性が維持されることを個別に検証する。

#### セレクタ


`data-animate` 属性および `data-stagger` 属性を使用する。

- `[data-animate="{種別}"]` — 要素自体のアニメーション種別を指定する（例: `data-animate="fade-in"`, `data-animate="slide-up"`）
- `[data-stagger="{種別}"]` — 子要素に段階的な遅延を適用するコンテナ。値はアニメーション種別で、`data-animate` と同じ `@keyframes` を共有する（例: `data-stagger="fade-in-slide-up"`）。JS が各子要素に `--stagger-delay` を設定する

> **設計根拠**: Animation 層のスタイルは JS（IntersectionObserver 等）と連動する。JS からの要素取得には `data-*` 属性を使用する（§6 JS 連携）ため、`data-animate` 属性セレクタを採用する。

#### 動きの分類

Animation 層に分離すべき動きと、Component/Project に残してよい動きを区別する。

| 分類 | 性質 | 例 | 層 |
|---|---|---|---|
| 装飾的アニメーション | 視覚演出。なくても機能に影響しない | fade-in, slide-up, stagger, parallax | Animation |
| 機能的トランジション | インタラクションフィードバック。ユーザー操作に対する応答 | hover の色変化, ボタンの translate, フォーカスリングの遷移 | Component / Project |

#### @keyframes 命名

`@layer` は `@keyframes` 名のスコープを分離しないため、属性値との対応関係を明確にすることで名前衝突のリスクを低減する（例: `data-animate="scale-in"` → `@keyframes scale-in`）。

> **Example（MUST [2 ガード原則の実装]、SHOULD [統合ガードの使用]、SHOULD [@keyframes 名と data-* 属性値の一致]）:**

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
    [data-animate="fade-in"][data-visible] {
      animation-play-state: running;
    }
  }
}
```

### 5.8 Utility

**責任**: 単一目的のスタイル上書き

**プレフィックス**: `u-`

**Utility 層の責任:**

1. **単一目的のスタイル上書き**: 1 プロパティ（または密接に関連する最小セット）で完結
2. **最終上書きの保証**: `!important` による全層に対する優先
3. **Block に帰属しないグローバルな補助**: 特定の Component に属さない汎用的なスタイル

**検証問い（Utility Test）**: 「このスタイルは特定の Block に帰属せず、単一プロパティ（または密接に関連する最小プロパティセット）で完結するか？」

- Yes → Utility
- No → Component / Project

**要求レベル:**

- MUST [!important の付与]: `!important` を付与しなければならない（使用制限については §4 を参照）
- SHOULD NOT [Block 帰属スタイルの Utility 記述禁止]: 特定の Block や Element に帰属できるスタイルを Utility に書くべきでない — Utility は特定の Block に帰属しない横断的かつ局所的な単一目的のスタイルに限る
- MAY [セマンティックなファイルグループ化]: セマンティックな意味でファイルをグループ化してよい（例: `u-hidden.css` に `u-visually-hidden` と `u-hidden-sp` をまとめる）

**適切な用途**: アクセシビリティ非表示（`u-visually-hidden`）、レスポンシブ表示制御

**不適切な用途**: 色の定義、レイアウトの組み立て、コンポーネントの装飾

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

層の識別・Block/Element/Modifier の構造・状態表現を体系化する命名規則を定義する。一貫した命名によりコードの可読性と保守性を高める。

**要求レベル:**

- MUST [Block なし Element の使用禁止]: Element（`__element`）は、対応する Block クラスが HTML 上に存在しなければならない。Block なしの Element は使用してはならない。CSS にルールセットがなくても、HTML 上に Block クラスが付与されていれば MUST 違反にはならない。BEM の「Element は常に Block の一部であり、Block から分離して使用してはならない」に基づく
- SHOULD [層識別プレフィックスの使用]: 層の識別のためにプレフィックス（§3 の層テーブルを参照）を使用すべきである。`@scope` 等の将来の CSS 機能によりプレフィックスが不要になる可能性があるため MUST としない。Token・Reset・Foundation はクラスセレクタを使用しないため、Animation は `data-animate` 属性セレクタを使用するため（§5.7 設計根拠を参照）、プレフィックスの対象外とする
- SHOULD NOT [Element の深いネスト禁止]: Element を 2 階層以上ネストすべきではない。Element が深くなる場合はコンポーネントの分離を検討する。
- SHOULD [Modifier の使用可能層]: Modifier は Component / Layout / Project で使用すべきである
- SHOULD NOT [Animation・Utility での Modifier 使用禁止]: Animation 層および Utility 層では Modifier を使用すべきではない
- SHOULD [ファイル名と主要クラス名の一致]: ファイル名は主要クラス名と一致させるべきである — `{prefix}-{name}.css`。1 ファイルに関連クラスをグループ化する場合は、代表クラス名をファイル名とする（例: `u-hidden.css`）。Animation 層は `{種別}.css`（例: `fade-in.css`）とし、`data-animate` の値に対応させる（`animation/` ディレクトリが層を識別するため、ファイル名にプレフィックスは不要）。

### クラス名

```
.{prefix}-{name}__{element}
.{prefix}-{name}.-{modifier}
```

| 要素 | 形式 | 例 |
|---|---|---|
| Block | `.{prefix}-{name}` | `.c-button`, `.l-section` |
| Element | `.{prefix}-{name}__{element}` | `.c-card__title` |
| Modifier | `.-{modifier}` | `.c-button.-primary`, `.l-section.-wide` |
| State（data-*） | `[data-{state}]` | `[data-active]`, `[data-loading]`, `[data-visible]` |
| State（ARIA） | `[aria-{prop}="..."]` | `[aria-expanded="true"]`, `[aria-current="page"]` |

### Modifier と State の使い分け

- **Modifier（`.-xxx`）**: 静的なバリエーション。Block または Element と併用する。HTML に記述し、原則として変化しない
- **State**: 要素の動的な状態。`data-*` 属性または ARIA 属性（`aria-expanded`, `aria-current` 等）で表現する。JS やユーザー操作により変化する。Block・Element いずれとも組み合わせて使用できる（例: `.c-button[data-loading]`、`.c-card__title[data-active]`）

### Modifier に `-` を採用する理由

BEM の `--` ではなく `.-modifier` を採用する。

- HTML がコンパクトになる（`class="c-button -primary"` vs `class="c-button c-button--primary"`）
- CSS Nesting との相性が良い

### Element の深さ

> **Example（SHOULD NOT [Element の深いネスト禁止]）:**

```css
/* 非推奨: Element のネスト */
.p-hero__content__title

/* 推奨: Element 名をフラットにする */
.p-hero__content-title

/* 推奨: 別の Block に分離する */
.p-hero-content        /* 新しい Block */
.p-hero-content__title /* その Element */
```

### JS 連携

`js-` プレフィックスは設けない。JS からの要素取得には `data-*` 属性または ARIA 属性を使用する。

### ファイル名

> **注記（Informative）**: 例: `.c-button` → `c-button.css`, `.l-header` → `l-header.css`, `.l-section` → `l-section.css`, `[data-animate="fade-in"]` → `animation/fade-in.css`

### カスタムプロパティ命名まとめ

| 層 | パターン | 例 |
|---|---|---|
| Token（プリミティブ） | `--_{カテゴリ}-{名前}` | `--_slate-600` |
| Token（セマンティック） | `--{カテゴリ}-{役割}` | `--color-main`, `--font-size-body` |
| Token（その他カテゴリ） | `--{カテゴリ}-{名前}` | `--space-md`, `--ease-out-cubic` |
| 公開 API | `--{対象}-{名前}` | `--section-padding`, `--badge-bg` |
| プライベートカスタムプロパティ | `--_{名前}` | `--_font-size-min` |

---

## 7. Custom Properties

*This section is normative.*

カスタムプロパティの参照チェーン・公開 API / プライベート命名規則・インラインスタイルの使用制限を定義する。

**要求レベル:**

- SHOULD [セマンティック変数経由の参照]: Foundation 以降の層はブランドトークンについて Token 層のセマンティック変数を参照すべきである
- SHOULD NOT [プリミティブ変数の直接参照禁止]: Foundation 以降の層がプリミティブ変数（`--_` プレフィックス）を直接参照すべきでない（セマンティック変数を経由する）
- MAY [グローバルトークンの直接参照]: グローバルトークン（ease, z-index 等）は Foundation 以降の層から直接参照してよい（Token 層のトークン分類を参照）
- SHOULD [公開 API 変数の命名規則遵守]: 上位層（Project 等）から値を設定する変数、または JS から値を注入する変数は、`--{対象}-{名前}` の公開 API 命名を使用すべきである。外部からの契約として機能する変数にプライベート命名（`--_`）を使用すると、意図が不明確になる
- MAY [プライベートカスタムプロパティの定義]: Layout 以降の層（Layout, Component, Project, Animation）でプライベートカスタムプロパティ（`--_xxx`）を定義してよい。プライベートカスタムプロパティは外部から参照・設定されることを想定しない内部実装である
- MUST NOT [静的インラインスタイルの禁止]: HTML マークアップに静的なインラインスタイルを記述してはならない。インラインスタイルはどの `@layer` にも属さない unlayered CSS として扱われ、全ての layered CSS より優先されるため、層構造による優先順位制御を破壊する
- MAY [JS からの動的カスタムプロパティ注入]: JS から公開 API のカスタムプロパティの値を動的に注入してよい。CSS が「何をするか」を定義し、JS が「いつ・どの値で」を決める互いに依存しない連携パターンとして許容する

### 参照チェーン

カスタムプロパティは以下の経路で使用する。

```
Token（プリミティブ → セマンティック）→ Foundation 以降（使用）
```

### 公開 API / プライベート命名規則

カスタムプロパティは公開 API とプライベートに分類する。

| 分類 | プレフィックス | 用途 | 例 |
|---|---|---|---|
| 公開 API | `--{対象}-{名前}` | 上位層または JS から上書きされる変数 | `--section-padding`, `--badge-bg`, `--stagger-delay` |
| プライベートカスタムプロパティ | `--_` | Block または Element 内部でのみ使用する変数 | `--_font-size-min`, `--_delay` |

上位層（Project 等）から公開 API の値を設定し、Layout や Component の振る舞いをページやセクションに合わせて変える。これは §5.6 の CSS 変数経由上書きパターンの基盤となる。

> **注記（Informative）**: 上位層（Project）が下位層（Layout, Component）の公開 API 変数の値を設定することは、正しい依存方向（上位→下位）に沿った操作であり、§3 の MUST NOT（下位→上位の参照禁止）に抵触しない。

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

### インラインスタイル

> **注記（Informative）**: MUST NOT [静的インラインスタイルの禁止] は開発者が意図的に記述するスタイルに適用される。CMS やライブラリが自動生成するインラインスタイルは適用対象外とする。

---

## 8. File Architecture

*This section is normative.*

ファイルとディレクトリの構造・命名・エントリポイントの構成を定義する。物理的なファイル配置と層構造の対応を規定する。

**要求レベル:**

- MUST [ディレクトリ名の層名一致]: ディレクトリ名は層名と一致させなければならない。
- SHOULD [1 Block = 1 ファイルの維持]: 1 つの CSS ファイルには 1 つの Block を定義すべきである。複数の独立した Block を 1 ファイルに含めると、ファイル名と Block の対応が崩れる。
- SHOULD [層帰属の明確化]: 各ファイルがどの層に属するかを明確にすべきである。方法はプロジェクトの規模やツールチェーンに応じて選択してよい

### ディレクトリ構造

> **Example（MUST [ディレクトリ名の層名一致]）:**

```
css/
├── layer-order.css
├── property.css          /* 任意（@property を使用する場合のみ） */
├── style.css
├── token/
├── reset/
├── foundation/
├── layout/
├── component/
├── project/
├── animation/
└── utility/
```

> **注記（Informative）**: `layer-order.css` は `@layer` の外で機能する宣言であり、いずれの層にも属さない。`property.css` は `@property` を使用する場合のみ作成する（§4 参照）。いずれも層ディレクトリと同列に配置する。

### 1 Block = 1 ファイル

> **注記（Informative）**: Utility 層は例外とする。Utility は Block 構造を持たない単一目的のクラスであり、セマンティックな意味でグループ化することが推奨される（§5.8 参照）。

### layer-order.css

`@layer` の先制宣言のみを含む。スタイル定義は一切置かない。

### property.css

`@property` 宣言を含む。§4 の MUST [@property のレイヤー外配置] により `@layer` の外に配置する必要があるため、層ディレクトリには入れない。`@property` を使用しないプロジェクトではこのファイルは不要である。

### style.css（エントリポイント）

> **注記（Informative）**: 層帰属の明確化方法はプロジェクトの規模やツールチェーンに応じて選択してよい。

| 方式 | 説明 |
|---|---|
| `@import layer()` | エントリポイントで層の帰属を一元管理する（推奨） |
| 層ごとの index.css | 各層の index.css 内で `@layer` ブロックを使ってまとめる |
| ビルドツール自動解決 | ビルドツールが glob import で自動的にファイルを収集する |

いずれの方式でも、ディレクトリ名・ファイルプレフィックス・クラスプレフィックスによって層の帰属は識別できる。

---

## Appendix A: Glossary

*This section is normative.*

| 用語 | 定義 |
|---|---|
| **先制宣言** | `layer-order.css` における `@layer` による層間の優先順位宣言。全スタイル定義に先行して記述される（初出: §4） |
| **Token Test** | Token 層の適用可否を判定する検証問い（§5.1） |
| **ブランドトークン** | プロジェクトごとに変わるデザイン値（color, typography, structure）。プリミティブ変数とセマンティック変数の総称。参照ルールは §5.1 および §7 を参照（初出: §5.1） |
| **プリミティブ変数** | `--_` プレフィックスを持つ Token 層の変数。生の値（HEX 色値、px 数値等）を保持する。ブランドトークンの一種（初出: §5.1） |
| **セマンティック変数** | 意味を持つ Token 層のマッピング変数（`--color-main` 等）。プリミティブ変数を参照し、コンテキスト（ダークモード等）に応じた役割を表現する。ブランドトークンの一種（初出: §5.1） |
| **グローバルトークン** | 多くのプロジェクトで共通して使える値（ease, z-index, font-weight）。参照ルールは §5.1 および §7 を参照（初出: §5.1） |
| **計算ヘルパー** | 単位変換・計算のためのユーティリティ変数。Token 層に配置する（初出: §5.1） |
| **Reset Test** | Reset 層の適用可否を判定する検証問い（§5.2） |
| **Foundation Test** | Foundation 層の適用可否を判定する検証問い（§5.3） |
| **Layout Test** | Layout 層の責任範囲を判定する検証問い（§5.4） |
| **Portability Test** | Component と Project の境界を判定する基準テスト（§5.5） |
| **Responsibility Test** | Portability Test の補助テスト。判断が曖昧な場合にのみ使用（§5.5） |
| **Animation Test** | Animation 層の適用可否を判定する検証問い（§5.7） |
| **装飾的アニメーション** | 視覚演出としての動き。無効化しても機能に影響しない。Animation 層に分離し、2 ガード原則を適用する（初出: §5.7） |
| **機能的トランジション** | インタラクションフィードバックとしての動き。ユーザー操作に対する応答であり、対象の Block が属する層（Component または Project）に記述する。`transform` を含む場合は `prefers-reduced-motion` ガードを適用する（初出: §5.7） |
| **2 ガード原則** | Animation 層で `prefers-reduced-motion` と `scripting` の 2 条件を考慮すること。推奨は `@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` による統合ガード（初出: §5.7） |
| **統合ガード** | 2 ガード原則の推奨実装パターン。`@media (prefers-reduced-motion: no-preference) and (scripting: enabled)` で Animation 層のスタイル全体をラップし、条件を満たさない場合にブロック全体を不適用にする方式（初出: §5.7） |
| **Utility Test** | Utility 層の適用可否を判定する検証問い（§5.8） |
| **Block** | BEM における独立した意味のあるエンティティ。プレフィックス付きクラス名（`.c-card`, `.p-hero` 等）で表現する（初出: §6） |
| **Element（`__element`）** | Block の一部。命名は `.{prefix}-{name}__{element}` の形式。Block なしでの使用禁止等の規範的定義は §6 を参照（初出: §6） |
| **Modifier（`.-xxx`）** | 静的なバリエーション。定義は §6 を参照（初出: §6） |
| **参照チェーン** | Token（プリミティブ → セマンティック）→ Foundation 以降（使用）。カスタムプロパティの参照パスを規定する（初出: §7） |
| **公開 API（カスタムプロパティ）** | `--{対象}-{名前}` 形式の変数。上位層または JS から上書きされることを想定する外部インターフェース（初出: §7） |
| **プライベートカスタムプロパティ** | `--_` プレフィックスを持つ変数。Block または Element 内部でのみ使用し、外部からの参照・設定を想定しない（初出: §7）。「プライベート変数」は本用語の略称 |

---

## Appendix B: References

### Normative References

*This section is normative.*

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
- **[CSS-CASCADE-5]** Atkins Jr., T.; Rivoal, F.; Lilley, C., "CSS Cascading and Inheritance Level 5", W3C Candidate Recommendation.
- **[CSS-PROPERTIES-VALUES-1]** Atkins-Bittner, T.; Stearns, A.; Whitworth, G., "CSS Properties and Values API Level 1", W3C Candidate Recommendation.

### Informative References

*This section is informative.*

- **[FLOCSS]** Hiloki, "FLOCSS — Foundation Layout Object CSS". https://github.com/hiloki/flocss
- **[BOOK]** shunei,『そのFLOCSS、なぜそこに書いた？』. https://zenn.dev/shunei/books/mflocss-design

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
| §4 | MUST [@property のレイヤー外配置] | §4 要求レベル |
| §4 | MUST NOT [!important の使用制限] | §4 要求レベル |
| §4 | MUST [外部 CSS の層取り込み] | §4 要求レベル |
| §5.1 | MUST [:root セレクタ限定] | §5.1 要求レベル |
| §5.1 | MUST NOT [他層カスタムプロパティ参照禁止] | §5.1 要求レベル |
| §5.2 | MUST [Reset 層の責任限定] | §5.2 要求レベル |
| §5.3 | MUST NOT [クラスセレクタ・ID セレクタの使用禁止] | §5.3 要求レベル |
| §5.7 | MUST [2 ガード原則の実装] | §5.7 要求レベル |
| §5.8 | MUST [!important の付与] | §5.8 要求レベル |
| §6 | MUST [Block なし Element の使用禁止] | §6 要求レベル |
| §7 | MUST NOT [静的インラインスタイルの禁止] | §7 要求レベル |
| §8 | MUST [ディレクトリ名の層名一致] | §8 要求レベル |

### SHOULD / SHOULD NOT

| § | 一言サマリ | 全文参照 |
|---|---|---|
| §5.1 | SHOULD [プリミティブとセマンティックの分離] | §5.1 要求レベル |
| §5.1 | SHOULD [テーマ切替の Token 完結] | §5.1 要求レベル |
| §5.2 | SHOULD [適切なリセット CSS の配置] | §5.2 要求レベル |
| §5.3 | SHOULD [属性セレクタの :where() 内使用] | §5.3 要求レベル |
| §5.3 | SHOULD [:where() による詳細度ゼロ] | §5.3 要求レベル |
| §5.3 | SHOULD [トークン参照の規則遵守] | §5.3 要求レベル |
| §5.3 | SHOULD [base と form の分離] | §5.3 要求レベル |
| §5.4 | SHOULD NOT [視覚プロパティの排除] | §5.4 要求レベル |
| §5.5 | SHOULD [Portability Test の合格] | §5.5 要求レベル |
| §5.5 | SHOULD NOT [サイト固有スタイルの排除] | §5.5 要求レベル |
| §5.5 | SHOULD [迷ったら Component 優先] | §5.5 要求レベル |
| §5.5 | SHOULD [トークン参照の規則遵守] | §5.5 要求レベル |
| §5.5 | SHOULD NOT [外部レイアウト影響プロパティの排除] | §5.5 要求レベル |
| §5.6 | SHOULD NOT [Component 相当スタイルの Project 記述禁止] | §5.6 要求レベル |
| §5.6 | SHOULD [セクションルートへの Project Block 付与] | §5.6 要求レベル |
| §5.6 | SHOULD [CSS 変数経由上書きの優先] | §5.6 要求レベル |
| §5.6 | SHOULD [ページ・機能単位のファイル分割] | §5.6 要求レベル |
| §5.7 | SHOULD [装飾的アニメーションの Animation 層分離] | §5.7 要求レベル |
| §5.7 | SHOULD [統合ガードの使用] | §5.7 要求レベル |
| §5.7 | SHOULD [transform を含む機能的トランジションのガード] | §5.7 要求レベル |
| §5.7 | SHOULD [@keyframes 名と data-* 属性値の一致] | §5.7 要求レベル |
| §5.8 | SHOULD NOT [Block 帰属スタイルの Utility 記述禁止] | §5.8 要求レベル |
| §6 | SHOULD [層識別プレフィックスの使用] | §6 要求レベル |
| §6 | SHOULD NOT [Element の深いネスト禁止] | §6 要求レベル |
| §6 | SHOULD [Modifier の使用可能層] | §6 要求レベル |
| §6 | SHOULD NOT [Animation・Utility での Modifier 使用禁止] | §6 要求レベル |
| §6 | SHOULD [ファイル名と主要クラス名の一致] | §6 要求レベル |
| §7 | SHOULD [セマンティック変数経由の参照] | §7 要求レベル |
| §7 | SHOULD NOT [プリミティブ変数の直接参照禁止] | §7 要求レベル |
| §7 | SHOULD [公開 API 変数の命名規則遵守] | §7 要求レベル |
| §8 | SHOULD [1 Block = 1 ファイルの維持] | §8 要求レベル |
| §8 | SHOULD [層帰属の明確化] | §8 要求レベル |

### MAY / MAY NOT

| § | 一言サマリ | 全文参照 |
|---|---|---|
| §3 | MAY [下位参照の許可] | §3 層間の依存方向 |
| §5.1 | MAY [カテゴリ別ファイル分割] | §5.1 要求レベル |
| §5.2 | MAY [Reset 層の使用任意] | §5.2 要求レベル |
| §5.4 | MAY [Container Queries 基盤の宣言] | §5.4 要求レベル |
| §5.4 | MAY [名前付きコンテナの定義] | §5.4 要求レベル |
| §5.5 | MAY [他層要素の内包] | §5.5 要求レベル |
| §5.6 | MAY [他層要素の内包] | §5.6 要求レベル |
| §5.6 | MAY [Layout と Component のみの構成] | §5.6 要求レベル |
| §5.6 | MAY [拡張用 Element クラスの先行付与] | §5.6 要求レベル |
| §5.7 | MAY [機能的トランジションの所属層への記述] | §5.7 要求レベル |
| §5.8 | MAY [セマンティックなファイルグループ化] | §5.8 要求レベル |
| §7 | MAY [グローバルトークンの直接参照] | §7 要求レベル |
| §7 | MAY [プライベートカスタムプロパティの定義] | §7 要求レベル |
| §7 | MAY [JS からの動的カスタムプロパティ注入] | §7 要求レベル |
