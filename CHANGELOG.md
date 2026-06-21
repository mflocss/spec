# Changelog

本書は mFLOCSS 仕様書の変更履歴を記録する。

## [v1.0] - Unreleased

初版リリース。

### Changed

- §4 に Informative 注記を追加: MUST [外部 CSS の層取り込み] が定める「いずれかの層」は §3 の 8 層に限定されず、役割をひとつに割り当てられない塊の第三者 CSS を取り込む任意の追加層（慣用名 `vendor`）を project が foundation より下位に宣言してよいことを明示。§3 の層構成および規範要求は不変。
- §6 SHOULD NOT [Element 併記の回避] を削除（業界標準 BEM / SUIT CSS / OOCSS と整合）
- §4 `!important` の要求レベルを統合再設計。MUST NOT [!important の使用制限] に対する許容範囲を「カスケードの勝敗を覆すためではなく、外せない保証・保護のためのみ」という判定軸で 3 類型（Utility 層の最終上書き保証 / 外部 CSS 取り込み時の制御外コードの非改変 / プラットフォーム不変条件の保護）に体系化。例外を任意層の外部 CSS 取り込みへ一般化し、MAY [プラットフォーム不変条件の保護] を新設して、ユーザーエージェントスタイルシート [HTML] が定める `display: none` の表示不変条件（状態限定）の保護に限り自作の `display: none !important` を許容
- §5.2 / §5.3: フォーム要素のフォント継承の正規化（`font: inherit`）の層帰属を整理。Reset Example（§5.2）に追加し、Foundation Example（§5.3）から除去
