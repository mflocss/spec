# Changelog

mFLOCSS 仕様の変更履歴。

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
