# mFLOCSS Specification

mFLOCSS（Modified FLOCSS）は、CSS の設計判断を体系化する思考フレームワークです。[FLOCSS](https://github.com/hiloki/flocss) の層設計を継承しつつ、`@layer` による構造的制御と明文化された判断基準を加えた 8 層フラットアーキテクチャを提供します。

このリポジトリは mFLOCSS の公式仕様書です。

## 仕様書

- [spec.md](spec.md) — mFLOCSS Specification v1.2
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

## 関連リソース

| リソース | 説明 |
|---|---|
| [FLOCSS](https://github.com/hiloki/flocss) | mFLOCSS の原典となった CSS 設計手法 |
| [書籍『そのFLOCSS、なぜそこに書いた？』](https://zenn.dev/shunei/books/mflocss-design) | 仕様の設計意図・判断プロセスの解説書 |
| [mflocss-starter](https://github.com/mflocss/starter) | リファレンス実装 |
| [mflocss.dev](https://mflocss.dev) | 公式サイト |

## 将来の展望

CSS の進化に応じて、以下の変更を検討する。

- **`@scope`**: 構造的スコープが普及した段階で、プレフィックス（`c-`, `p-`）の代替手段として仕様に組み込む可能性がある。現時点では `@scope` とプレフィックスの併用を推奨する
- **Animation 層の統合**: Scroll-driven Animations 等の進化により、Animation 層のスタイルが各層に統合される可能性がある

## ブランドガイドライン

「mFLOCSS」の名称を使用して仕様への準拠を表明する場合は、本仕様の全 MUST / MUST NOT ルールに準拠しなければなりません。準拠しない場合は「mFLOCSS」の名称を使用しないでください。

本仕様を改変して派生フレームワークを公開する場合は、「mFLOCSS」とは異なる名称を使用し、CC BY-SA 4.0 の条件（著作者表示・同一ライセンス）に従ってください。

## ライセンス

[CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — 著作者表示と同一ライセンスでの配布を条件に、商用利用・改変が可能です。
