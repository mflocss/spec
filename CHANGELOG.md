# Changelog

mFLOCSS 仕様の変更履歴。

> v1.0 は内部ドラフト。v1.1 が初の正式リリース。

## v1.1（初の正式リリース）

v1.0 内部ドラフトからの全変更を含む。

### 構造・方針

- §1.1 Introduction: 思考フレームワークの定義を厳密化（「ルールブックではなく」→「判断基準を厳密に定義する」、MUST の適用範囲を「思想の核」→「構造的整合性を保証するルール」に明確化）
- §1 のサブセクション化: Introduction [Informative], 不変原則 [Normative], 設計制約 [Normative] に分離
- §1.3 設計制約の新設: 一方向依存ルールを不変原則を支える構造的制約として規定。MUST NOT（§3 参照）を付記
- Normative / Informative ラベルの付与（W3C パターン）
- Example マーキングの付与
- §2 準拠条件: 設計判断を伴うルールの判定基準（テスト参照）を追記

### 層判断

- §3 Responsibility Test の追加: Portability Test を補助する判定テスト
- §3 Layer Judgment Flowchart の本文統合: 旧 Appendix A を §3 に移動し、誤りパターンを追加
- §3 依存方向: CSS の参照方向に限定し、HTML の入れ子構造はルール対象外と明確化

### 層定義

- §4 外部 CSS の層配置セクションの追加
- §4 CSS Cascading and Inheritance Level 5 への明示的参照の追加
- §5.1 トークン分類（ブランドトークン / グローバルトークン）の導入
- §5.2 Theme: セレクタ制約の追加（`:root` 基本 SHOULD、`data` 属性セレクタ MAY、`light-dark()` SHOULD）
- §5.3 Foundation: 「要素セレクタのみ」を「要素型セレクタのみ」に厳密化し、属性セレクタ・擬似クラス・擬似要素の許容範囲を明記
- §5.3 Foundation の `:where()` 詳細度ゼロを SHOULD に変更（外部リセット CSS のアタッチメント方式を考慮）
- §5.3 Foundation にサブレイヤー構成（foundation.reset / foundation.base）を SHOULD で追加
- §5.4 Container Queries の層責任テーブルを §9 から移動・統合（container-name ガイダンスを追加）
- §5.5 「迷ったら Component」の SHOULD 化と条件付き限定
- §5.5 Component: MUST NOT の例外（内部 Element）を構造的に分離
- §5.6 Project: Portability Test 合格スタイルの Project 記述を禁止する MUST NOT を追加
- §5.6 Project: セクションルートへの Project Block 付与を SHOULD で推奨
- §5.6 上書きパターンの拡充（CSS 変数経由パターンの追加）。Modifier 行に層の帰属を注記
- §5.7 Animation: 2 ガード原則の MUST を達成条件ベースに書き直し。統合ガードの SHOULD 化
- §5.8 Utility: MUST NOT の判定基準を「Block/Element への帰属可否」に明確化。`!important` MUST NOT の重複を §4 への参照に変更
- §9 Container Queries を SHOULD で規定

### 命名規則

- §6 ファイル名の MUST を SHOULD に変更（運用規約であり思想の核ではない）
- §6 プレフィックスの SHOULD を明記（@scope 等の将来の CSS 機能で不要になりうるため MUST としない）
- §6 Modifier の適用可能な層を明記（Component / Project で SHOULD、Layout / Animation / Utility で SHOULD NOT）
- §6 Block なしの Element 使用を禁止する MUST を追加（BEM の公式定義に基づく）
- §6 Element-Block MUST の「定義」を明確化: Block クラスの HTML 上の存在を要件とし、CSS ルールセット不在時の MUST 違反にならない旨を追記

### カスタムプロパティ

- §7 プライベートカスタムプロパティの上位層設定許容の明確化

### ファイル構成

- §8 ディレクトリ構造の簡素化（ファイル例の削除）

### Appendix

- References セクションの新設（Appendix B）
- Glossary に各用語の初出セクション番号を付記
- Glossary: Responsibility Test の定義文を更新（Portability Test との優先関係を明記）
- Glossary: Modifier の定義を追加
- Glossary: Block / Element の定義を追加
- Glossary: Element 定義から MUST を除去し、規範的定義は §6 参照に変更
- Changes を CHANGELOG.md に分離
- Appendix C: 不要なテーブル行を削除

### v1.0 からの削除

- 書籍参照（`→ 書籍 chX`）を本文から削除（Appendix B の Informative References に集約）
- カラートークンの階層化参考（§5.1）
- style.css のインポート例と注記（§8）
- コンポーネント追加/削除手順（§8）
- 将来の展望セクション（§1）

### 章番号対応表（v1.0 → v1.1）

| v1.0 | v1.1 | 内容 |
|---|---|---|
| §1 Introduction | §1 Overview | 親セクション名を変更。§1.1/§1.2/§1.3 にサブセクション化 |
| — | §1.1 Introduction [Informative] | 旧 §1 の導入部分 |
| — | §1.2 不変原則 [Normative] | 旧 §1 の不変原則部分 |
| — | §1.3 設計制約 [Normative] | v1.1 で新設 |
| §2 Conformance | §2 Conformance | 変更なし |
| §3 Layer Architecture | §3 Layer Architecture | Responsibility Test, Flowchart, 誤りパターン追加 |
| §4 Layer Order Declaration | §4 Layer Order Declaration | 外部 CSS の層配置追加 |
| §5 Layer Definitions | §5 Layer Definitions | 各層の強化 |
| §6 Naming Conventions | §6 Naming Conventions | 変更なし |
| §7 Custom Properties | §7 Custom Properties | 上位層設定許容の明確化 |
| §8 File Architecture | §8 File Architecture | 簡素化 |
| §9 Responsive Strategy | §9 Responsive Strategy | Container Queries テーブルを §5.4 に統合 |
| §10 Appendix A: Flowchart | — | §3 に統合、削除 |
| §11 Appendix B: Glossary | Appendix A: Glossary | 繰り上げ、初出番号付記 |
| — | Appendix B: References | 新設 |
