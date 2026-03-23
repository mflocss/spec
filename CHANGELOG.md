# Changelog

mFLOCSS 仕様の変更履歴。

## v1.0 → v1.1 の変更点

### 主要な変更

- §1 のサブセクション化: Introduction [Informative], 不変原則 [Normative], 設計制約 [Normative] に分離
- §1.3 設計制約の新設: 一方向依存ルールを不変原則を支える構造的制約として規定
- §3 Responsibility Test の追加: Portability Test を補助する判定テスト
- §3 Layer Judgment Flowchart の本文統合: 旧 Appendix A を §3 に移動し、誤りパターンを追加
- §3 依存方向: CSS の参照方向に限定し、HTML の入れ子構造はルール対象外と明確化
- §4 外部 CSS の層配置セクションの追加
- §4 CSS Cascading and Inheritance Level 5 への明示的参照の追加
- §5.1 トークン分類（ブランドトークン / グローバルトークン）の導入
- §5.4 Container Queries の層責任テーブルを §9 から移動・統合（container-name ガイダンスを追加）
- §5.4 に container-name の MAY ルールを新設
- §5.5 「迷ったら Component」の SHOULD 化と条件付き限定
- §5.6 上書きパターンの拡充（CSS 変数経由パターンの追加）
- §5.7 統合ガードの SHOULD 化と説明の簡潔化
- §5.8 u-visually-hidden の論理プロパティ対応
- §7 プライベートカスタムプロパティの上位層設定許容の明確化
- §8 ディレクトリ構造の簡素化（ファイル例の削除）
- Normative / Informative ラベルの付与（W3C パターン）
- Example マーキングの付与
- References セクションの新設（Appendix B）
- Glossary に各用語の初出セクション番号を付記
- Glossary: Responsibility Test の定義文を更新（Portability Test との優先関係を明記）
- §2 準拠条件: 設計判断を伴うルールの判定基準（テスト参照）を追記
- §5.3 Foundation: 「要素セレクタのみ」を「要素型セレクタのみ」に厳密化し、属性セレクタ・擬似クラス・擬似要素の許容範囲を明記
- §5.3 Foundation の `:where()` 詳細度ゼロを MUST から SHOULD に変更（外部リセット CSS のアタッチメント方式を考慮）
- §5.5 Component: MUST NOT の例外（内部 Element）を構造的に分離
- §5.5 Component: サイト固有禁止ルールの MUST → MUST NOT 修正（RFC 2119 準拠）
- §5.7 Animation: 2 ガード原則の MUST を達成条件ベースに書き直し
- §5.8 Utility: MUST NOT の判定基準を「Block/Element への帰属可否」に明確化
- §9 Container Queries を SHOULD で規定
- §5.3 Foundation にサブレイヤー構成（foundation.reset / foundation.base）を SHOULD で追加

### v1.1 内の変更（2026-03-23）

- §1.1 Introduction: 思考フレームワーク方針の明記（MUST は思想の核に限定、実践推奨は SHOULD）
- §5.2 Theme: セレクタ制約の追加（`:root` 基本、`data` 属性セレクタ許容）
- §5.8 Utility: `!important` MUST NOT の重複を §4 への参照に変更
- §6 Naming Conventions: ファイル名の MUST を SHOULD に変更
- §6 Naming Conventions: プレフィックスの SHOULD を明記
- §6 Naming Conventions: Modifier の適用可能な層を明記（Component / Project）
- §5.6 Project: Portability Test 合格スタイルの Project 記述を禁止する MUST NOT を追加
- §5.2 Theme: セレクタ制約の「許容する」に MAY を明記
- §6 Modifier: Layout/Animation/Utility での不使用を SHOULD NOT で明記
- §5.6 上書きパターン表: Modifier 行に層の帰属を注記
- §1.3: 依存方向の「禁止する」に MUST NOT（§3 参照）を付記
- Glossary: Modifier の定義を追加

### 削除された内容

- 書籍参照（`→ 書籍 chX`）を本文から削除（Appendix B の Informative References に集約）
- カラートークンの階層化参考（§5.1）
- style.css のインポート例と注記（§8）
- コンポーネント追加/削除手順（§8）
- 将来の展望セクション（§1）

### 章番号対応表

| v1.0 | v1.1 | 内容 |
|---|---|---|
| §1 Introduction | §1 Overview | 親セクション名を変更。§1.1/§1.2/§1.3 にサブセクション化 |
| — | §1.1 Introduction [Informative] | 旧 §1 の導入部分 |
| — | §1.2 不変原則 [Normative] | 旧 §1 の不変原則部分 |
| — | §1.3 設計制約 [Normative] | v1.1 で新設 |
| §1 不変原則（小見出し） | §1.2 不変原則 | サブセクションに昇格 |
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
