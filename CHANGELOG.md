# Changelog

本書は mFLOCSS 仕様書の変更履歴を記録する。

## [v1.0] - Unreleased

初版リリース。

### Changed

- §6 SHOULD NOT [Element 併記の回避] を削除（cortex#872 Option δ 採用 — 業界標準 BEM/SUIT CSS/OOCSS と整合、副作用 3.5/4 解消）
- §4 `!important` 要求レベルを統合再設計（cortex#1214）。抽象原則（カスケードの勝敗を覆すためでなく外せない保証・保護のためのみ許容）+ 許容 3 類型に体系化。例外を「Reset 層の外部リセット限定」から「任意層の外部 CSS 取り込み」へ一般化（Gap C）。MAY [プラットフォーム不変条件の保護] を新設し `[hidden]` の `display: none !important` を列挙的に許容（Gap A）
- §5.3 Foundation Example からフォーム要素の `font: inherit` を削除し、Reset 層（§5.2）が canonical 配置であることを Informative 注記で明示（Gap B、starter reference 実装と整合）
