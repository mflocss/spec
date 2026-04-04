# mFLOCSS Specification

mFLOCSS（Modified FLOCSS）は、CSS の設計判断を体系化する思考フレームワークです。[FLOCSS](https://github.com/hiloki/flocss) の層設計を継承しつつ、`@layer` による構造的制御と明文化された判断基準を加えた 8 層フラットアーキテクチャを提供します。

このリポジトリは mFLOCSS の公式仕様書です。

## 仕様書

- [spec.md](spec.md) — mFLOCSS Specification
- [CHANGELOG.md](CHANGELOG.md) — 変更履歴

## 仕様の読み方

本仕様のキーワード MUST / MUST NOT / SHOULD / SHOULD NOT / MAY は [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) に基づく。

仕様書内の要素は以下のように区別される:

| 要素 | 説明 |
|---|---|
| **Normative** | MUST / MUST NOT / SHOULD / SHOULD NOT / MAY を含むルール。準拠判定の対象 |
| **Informative** | 判断基準の解説、設計意図の説明。ルールの理解を助けるが準拠判定の対象外 |
| **Example** | コード例。ルールの具体的な適用方法を示す |
| **Changes** | バージョン間の変更履歴 |

## v1.0 の変更概要

書籍『そのFLOCSS、なぜそこに書いた？』との整合を目的とした変更。

### 文書構造の変更

| 変更 | 内容 |
|---|---|
| **Abstract / Status of This Document の追加** | 仕様書冒頭に概要セクションと文書ステータス（ドラフト）を追加 |
| **§9 Responsive Strategy の削除** | Container Queries の責任テーブルを §5.4 Layout に統合し、章として独立させない |
| **文書構造の統一** | §4・§6・§7・§8 を §5 と同じ章構造（責任→要求レベル→補足→Example）に再配置 |
| **要求レベルへの一言サマリ付与** | 全 MUST / MUST NOT / SHOULD / SHOULD NOT / MAY に角括弧で判定基準のラベルを付与（例: `MUST [先制宣言の実施]`） |
| **Appendix D: Requirements Index の追加** | 仕様内の全要求レベルを章番号・一言サマリ付きで一覧化 |

### 仕様内容の変更

| 変更 | 内容 |
|---|---|
| **State の表現方法** | `.is-{state}` クラスを廃止し、`data-*` 属性または ARIA 属性（`aria-expanded` 等）で状態を表現する |
| **§2 準拠条件の MUST 基準を明確化** | MUST の使用範囲をカスケード保護・依存方向保護・Utility フェイルセーフの 3 種に限定して明文化 |
| **§3 上位/下位層の定義追加** | `@layer` 優先度の文脈で「上位層」「下位層」の用語定義を追記（Utility が最上位、Token が最下位） |
| **style.css の MUST を SHOULD に降格** | `style.css` のエントリポイント構成をプロジェクト運用の判断に委ねる SHOULD に変更 |
| **§6 を BEM 差分定義に集中** | BEM 一般原則（Block / Element / Modifier の基本定義）の MUST / SHOULD を削除し、mFLOCSS 独自の拡張差分のみを規定 |
| **§6 Element 連鎖を SHOULD NOT に降格** | Element のネスト（`.c-card__body__title` 等）を MUST NOT から SHOULD NOT に変更 |
| **§7 トークン参照チェーンの体系化** | 章タイトルを「Custom Properties」に変更し、プリミティブ→セマンティック→コンポーネント変数の参照チェーンを明文化 |
| **§7 セマンティック変数の定義緩和** | セマンティック変数がプリミティブ変数を参照することを SHOULD（推奨）に変更し、直接値の定義も許容 |
| **ライセンスを CC BY-SA 4.0 に変更** | CC BY-ND 4.0 から CC BY-SA 4.0 へ変更（改変・再配布を著作者表示と同一ライセンスの条件で許可） |

## 関連リソース

| リソース | 説明 |
|---|---|
| [FLOCSS](https://github.com/hiloki/flocss) | mFLOCSS の原典となった CSS 設計手法 |
| [書籍『そのFLOCSS、なぜそこに書いた？』](https://zenn.dev/shunei/books/mflocss-design) | 仕様の設計意図・判断プロセスの解説書 |
| [mflocss-starter](https://github.com/mflocss/starter) | リファレンス実装 |
| [mflocss.dev](https://mflocss.dev) | 公式サイト |

## 将来の展望

CSS の進化に応じて、以下の変更を検討する。

- **`@scope`**: 構造的スコープが普及した段階で、プレフィックス（`c-`, `p-`）の代替手段として仕様に組み込む可能性がある。現時点ではプレフィックスを使用する（§6 SHOULD [層識別プレフィックスの使用]）
- **Animation 層の統合**: Scroll-driven Animations 等の進化により、Animation 層のスタイルが各層に統合される可能性がある

## ブランドガイドライン

「mFLOCSS」の名称を使用して仕様への準拠を表明する場合は、本仕様の全 MUST / MUST NOT ルールに準拠しなければなりません。準拠しない場合は「mFLOCSS」の名称を使用しないでください。

本仕様を改変して派生フレームワークを公開する場合は、「mFLOCSS」とは異なる名称を使用し、CC BY-SA 4.0 の条件（著作者表示・同一ライセンス）に従ってください。

## ライセンス

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC_BY--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)

[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — 著作者表示と同一ライセンスでの配布を条件に、商用利用・改変が可能です。
