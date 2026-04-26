# Changelog

mFLOCSS 仕様の変更履歴。

## [v1.0] - Unreleased

Initial Release.

- §5.7 SHOULD [Animation 層公開 API の概念名命名] を追加
- §7 MAY [公開 API 変数への @property 宣言] を追加（v1.0 純化により SHOULD から MAY に降格）

### v1.0 純化

業界スタンダード候補化のため、判断フロー / 実装パターン Example を spec から逃がし、要求レベルの純粋な列挙に整理した。判断フロー・既存手法比較・実務応用は書籍へ、実装パターン Example はリファレンス実装（starter）+ agent reference へ移設する（[knowledge#3 book-spec-separation](https://github.com/shunei-web/knowledge/pull/3)）。

- §5.5 / §5.6 Informative を削除（外部レイアウト影響適用範囲、overflow 判断フロー、Project 上書きパターン表、Modifier vs Project 上書き境界判定）
- §6 Informative「Modifier 記法の選択根拠は書籍参照」を削除（原典が非原典を参照する倒置の解消）
- §7 Informative「@property はネイティブ CSS 機能」を削除
- §7 Example 3 件を削除（`.c-card` + `.p-about`、公開 API 命名 NG/OK 対比、`@property` 宣言）
- §7 SHOULD [公開 API 変数への @property 宣言] を MAY に降格
