# Contributing to mFLOCSS Specification

mFLOCSS 仕様書（[spec.md](spec.md)）への提案・修正を歓迎します。

## 提案・修正の流れ

- 仕様の変更提案・誤りの指摘・曖昧さの報告は Issue で議論してください。
- 規範要求（MUST / MUST NOT / SHOULD / SHOULD NOT / MAY）の追加・変更・削除を伴う提案は、影響範囲（既存の準拠実装・リファレンス実装・書籍）を明記してください。
- 合意された変更は PR で反映します。

## 変更履歴（CHANGELOG）の運用

- バージョンは spec.md §2「バージョニング」に定める「メジャー.マイナー.パッチ」の3桁で表記します。
- CHANGELOG は、前回リリース以降の変更をリリース対象バージョンの `## [vX.Y.Z] - Unreleased` セクションに追記していきます。
- 初版（v1.0）は比較対象の前バージョンが存在しないため変更履歴を持たず、`初版リリース。` のみとします。

## リリース手順

1. spec.md の「Status of This Document」を更新（版の表記と「最終更新」の日付）
2. リリースするバージョンの `## [vX.Y.Z] - Unreleased` 見出しを `## [vX.Y.Z] - YYYY-MM-DD` に置換（リリース実日付を記入）
3. `git tag vX.Y.Z` でリリースタグを切る
4. 次バージョンの変更を記録し始める際に、新しい `## [vX.Y.Z] - Unreleased` セクションを CHANGELOG 上部に追加

## ライセンス・ブランド

本仕様は [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) の下で提供されます。「mFLOCSS」名称の使用条件・派生フレームワークの扱いは [README.md](README.md#ブランドガイドライン) を参照してください。
