# Changelog

本書は mFLOCSS 仕様書の変更履歴を記録する。

## [v1.0] - Unreleased

初版リリース。

### Changed

- §5.2 / §5.7 / §5.8 / §7 / §8 から実装レベル防御策・運用ノウハウ規定 8 件を削除（業界スタンダード候補化のための純度純化、cortex#468 観点 4）
  - §5.2 SHOULD [flex / Grid item の content-based minimum size 罠の予防] → 実装詳細（agent-reference 移管候補）
  - §5.7 SHOULD [2 ガード原則の実装] → 実装ガード仕様（agent-reference 移管候補）
  - §5.7 SHOULD [translate / rotate / scale を含む機能的トランジションのガード] → 特定 CSS プロパティ列挙（agent-reference 移管候補）
  - §5.8 MAY [セマンティックなファイルグループ化] → ファイル管理運用（agent-reference 移管候補）
  - §7 MAY [公開 API 変数への @property 宣言] → CSS API 使用許可（agent-reference 移管候補）
  - §8 SHOULD [ディレクトリ名の層名一致] → ディレクトリ運用（agent-reference 移管候補）
  - §8 SHOULD [1 Block = 1 ファイルの維持] → ファイル管理運用（agent-reference 移管候補）
  - §8 SHOULD [層帰属の明確化] → 運用ノウハウ（agent-reference 移管候補）
- §8 File Architecture 章を完全削除（純度棚卸し v1.0、規範規定ゼロのため章として存在意義喪失、cortex#468 観点 4 / しゅんえい A 案）
  - 前変更で §8 SHOULD 3 件すべて削除済 → 規範規定がゼロとなり「This section is normative」を維持不能
  - 残存内容は layer-order.css / property.css の Informative 注記 2 つのみ（注記も削除）
  - layer-order.css 情報は §4 MUST [先制宣言の実施] で間接表現済
  - property.css 情報は agent-reference / 書籍（ch11）が担当
- Appendix B: Informative References から [CSS-FLEXBOX-1] / [CSS-GRID-1] を削除（dangling reference 除去）
- Appendix D: 削除規定の Requirements Index 行を除去
