# Changelog

mFLOCSS 仕様の変更履歴。

## [v1.1] - 2026-04-02

### 仕様書構造の統一（§4・§6・§7・§8）

§5 Layer Definitions と同じ章構造（章の目的→要求レベル→補足→Example→誤りパターン）に §4・§6・§7・§8 を再配置。各章冒頭に責任/目的を追記。MUST/SHOULD/MAY を各章内の **要求レベル:** セクションにまとめ、解説・補足は Informative として残す。内容は変更しない（構造の再配置のみ）。

### Abstract セクション追加

仕様書冒頭に Abstract セクションを追加。mFLOCSS の概要を非技術者にも分かる形で 3 文にまとめる。

### Status of This Document セクション追加

冒頭のメタデータ（ステータス: ドラフト）を W3C 仕様の Status of This Document 相当のセクションに格上げ。ドラフト版であること・v1.0 で正式版になる予定を明記。

### Example キャプション付与（全 §）

§4・§5・§6・§7・§8 の全 Example と誤りパターン（コード例）に、示す要求レベルの一言サマリをキャプションとして付与。

### Appendix D: Requirements Index 追加

spec 内の全 MUST/MUST NOT/SHOULD/SHOULD NOT/MAY を章番号・一言サマリ付きで一覧化。各項目は本文の該当箇所への参照を含む。

---

## [v1.0] - 2026-04-02

Initial Release。

### State の表現方法

`.is-{state}` クラスを廃止。状態は `data-*` 属性または ARIA 属性（`aria-expanded`, `aria-current` 等）で表現する。JS との連携に `data-*` 属性を統一使用することで、クラスと属性の役割を明確に分離する。

### §9 Responsive Strategy の削除

Container Queries の層責任テーブルを §5.4 Layout に統合し、§9 を削除。関連ルールを Layer Definitions 内に集約する。

### 要求レベルへの一言サマリ付与

全 MUST / MUST NOT / SHOULD / SHOULD NOT / MAY に角括弧で判定基準のラベルを付与（例: `MUST [Block なしの Element 使用禁止]`）。仕様の走査性と準拠判定を向上させる。

### style.css の MUST を SHOULD に降格

`style.css` のエントリポイント構成ルールを SHOULD に変更。ファイル構成はプロジェクトの運用規約であり、仕様の核となる構造的整合性ルールではないため。

### Modifier の Layout 使用可を明記

Modifier を使用できる層に Layout を追加。Component / Layout / Project で SHOULD、Animation / Utility で SHOULD NOT と明記する。

### Element 形式の完全化

Element 命名を `.{prefix}-{name}__{element}` 形式に統一。Block クラスが HTML 上に存在しない状態での Element 単独使用を MUST で禁止する（BEM の公式定義に準拠）。

### プライベート変数の Element 定義許容

プライベートカスタムプロパティ（`--_xxx`）を Element スコープで定義できることを §7 に明記。Block 内部の計算用変数を Element ルールセット内で宣言するユースケースを許容する。
