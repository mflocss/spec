# mFLOCSS Specification

![mFLOCSS — CSS の設計判断を体系化する思考フレームワーク](.github/ogp.png)

mFLOCSS は、CSS の記述を層に分類する思考フレームワークです。「このスタイルをどの層に書くか」という判断を体系化し、設計の一貫性と長期的な保守性を実現します。本仕様はその判断基準・命名規則・ファイル構成を厳密に定義します。

このリポジトリは mFLOCSS の公式仕様書です。

## 仕様書

- [spec.md](spec.md) — mFLOCSS Specification
- [CHANGELOG.md](CHANGELOG.md) — 変更履歴

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
