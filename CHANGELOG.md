# Changelog

mFLOCSS 仕様の変更履歴。

## [v1.0] - Unreleased

Initial Release。書籍『そのFLOCSS、なぜそこに書いた？』との整合を目的とした変更を含む。

### 文書構造の変更

#### Abstract / Status of This Document の追加

仕様書冒頭に Abstract（概要）セクションと Status of This Document（文書ステータス）セクションを追加。W3C 仕様の慣行に倣い、対象読者・ドラフトステータス・最終更新日を明記する。

#### §9 Responsive Strategy の削除

Container Queries の層責任テーブルを §5.4 Layout に統合し、§9 を削除。関連ルールを Layer Definitions 内に集約することで章間の重複を解消する。

#### 文書構造の統一（§4・§6・§7・§8）

§5 Layer Definitions の章構造（責任の定義→要求レベル→補足→Example→誤りパターン）に §4・§6・§7・§8 を統一。各章冒頭に責任/目的を明記し、MUST/SHOULD/MAY を **要求レベル:** セクションにまとめる。内容は変更しない（構造の再配置のみ）。

#### 要求レベルへの一言サマリ付与

全 MUST / MUST NOT / SHOULD / SHOULD NOT / MAY に角括弧で判定基準のラベルを付与（例: `MUST [先制宣言の実施]`）。仕様の走査性と準拠判定を向上させる。全 §5 の Example にも要求レベルのキャプションを付与。

#### Appendix D: Requirements Index の追加

spec 内の全 MUST / MUST NOT / SHOULD / SHOULD NOT / MAY を章番号・一言サマリ付きで一覧化。各項目は本文の該当箇所への参照を含む。

---

### 仕様内容の変更

#### State の表現方法

`.is-{state}` クラスを廃止。状態は `data-*` 属性または ARIA 属性（`aria-expanded`, `aria-current` 等）で表現する。JS との連携に `data-*` 属性を統一使用することで、クラスと属性の役割を明確に分離する。

#### §2 準拠条件の MUST 基準を明確化

MUST の使用範囲を以下の 3 種に限定して明文化した。

1. `@layer` の層構造カスケードを保護するルール
2. 層間の依存方向を保護するルール
3. Utility 層の最終上書きを構造的に保証するルール（他ルール違反時のフェイルセーフ）

設計判断の品質（層の選択・Portability Test 等）は SHOULD で推奨する。

#### §3 上位/下位層の定義追加

「上位層」「下位層」を `@layer` 優先度の文脈で定義する記述を追加。順序番号が大きい層（優先度が高い）を上位、小さい層を下位と定義する（例: Utility（8）が最上位、Token（1）が最下位）。

#### style.css の MUST を SHOULD に降格

`style.css` のエントリポイント構成ルールを SHOULD に変更。ファイル構成はプロジェクトの運用規約であり、仕様の核となる構造的整合性ルールではないため。

#### §6 を BEM 差分定義に集中

Block / Element / Modifier の基本定義に関する BEM 一般原則の MUST / SHOULD（4 件）を §6 から削除。§6 は mFLOCSS 独自の拡張差分（プレフィックス規則・Modifier の層制約・State の表現方法等）のみを規定する構成に集約した。

#### §6 Element 連鎖を SHOULD NOT に降格

Element のネスト（`.c-card__body__title` 等）を MUST NOT から SHOULD NOT に変更。BEM 公式でも SHOULD NOT 相当の推奨であり、実務での例外ユースケースを考慮した。

#### §7 トークン参照チェーンの体系化

§7 の章タイトルを「Custom Properties」に統一し、プリミティブ変数→セマンティック変数→コンポーネント変数への参照チェーンを明文化。インラインスタイルの MUST NOT 禁止ルールから動的・自動生成の除外条件を本文に統合し、要求レベルを簡潔化。

#### §7 セマンティック変数の定義緩和

セマンティック変数がプリミティブ変数を参照することを SHOULD（推奨）に変更し、直接値での定義も許容。厳格な参照強制ではなく、段階的な採用を可能にする。

#### ライセンスを CC BY-SA 4.0 に変更

CC BY-ND 4.0 から CC BY-SA 4.0 へ変更。改変・再配布を著作者表示と同一ライセンスの条件で許可し、派生フレームワークの公開を可能にする。
