# AR レポート: cortex#1214 `!important` 要求レベル統合再設計

- 対象 PR ブランチ: `fix/important-redesign-1214`
- ベース: `fix/v1.0-book-alignment`
- レビュー実施: 2026-06-07
- レビュアー: mflocss-spec-lead（独立 AR）
- レビュー対象差分: spec.md（§4 / §5.3 / Requirements Index）+ CHANGELOG.md

## 結論

**PASS**（要修正なし）。spec の純度・整合・列挙閉性・Reset canonical 整合・バージョニングのすべての観点で問題なし。

## 評価軸

### 1. 純度（追加 Informative が why/how 過多でないか）

| 追加箇所 | 内容 | 判定 |
|---|---|---|
| §4「例外: `!important` の許容範囲」本文 | 三類型を列挙し、各類型が spec 内の MUST / MUST NOT / MAY とどう接続するか（cross-reference）を述べる | **PASS**: spec 内クロスリファレンスは規範文の整合性記述であり、判断フロー（how）でも背景説明（why）でもない。純度違反兆候の「判断フロー 3 段以上」「具体パターン名 4 例以上」「既存手法比較」のいずれにも該当しない。 |
| §4 注記（Informative）「三類型に共通する判定軸は…」 | 抽象原則の判定軸を 1 文で示す + 将来 `@scope` 普及時の見直し可能性 1 文 | **PASS**: 判定軸は規範の **意図** を 1 行で要約しているのみで、判断フロー（複数段）や具体手法解説ではない。`@scope` 注記は将来の規範変更の可能性 1 文のみで how/why 詳述ではない。 |
| §5.3 Foundation Example 直後の注記 | フォーム要素の `font: inherit` を Reset 層に配置する canonical 判断を 1 文で示す | **PASS**: 「Reset Test に該当するため §5.2」という規範間の責任境界記述。実装パターン解説 / Example 追加ではない。 |

> 注: 抽象原則「カスケードの勝敗を覆すためか、外せない保証・保護のためか」が本文（規範）に格上げされた点は **純度向上**。従来は「Reset 内部実装」「Utility」「外部 CSS」と個別列挙的に散らばっていた MUST NOT 例外境界が、規範本文の三類型として体系化された。

### 2. 整合（§4 / §5.8 / §5.3 / Requirements Index / §2 Conformance 准拠条件 の相互参照）

| 参照ペア | 整合性 |
|---|---|
| §4 MUST NOT [!important の使用制限] ⇄ §5.8 MUST [!important の付与] | **整合**。§5.8 MUST 注記 L639「使用制限については §4 を参照」と新 §4 三類型 type 1 が双方向参照を構成。§5.8 の文言変更不要。 |
| §4 MUST NOT ⇄ §4 MAY [プラットフォーム不変条件の保護] | **整合**。MUST NOT 本文の「ただし『例外: `!important` の許容範囲』に定める場合を除く」が MAY との関係を明示し、三類型 type 3 で MAY を再参照。循環の問題なし。 |
| §4 三類型 type 2 ⇄ §4 MUST [外部 CSS の層取り込み] | **整合**。type 2 が MUST [外部 CSS の層取り込み] を明示参照し、Cat2 が Reset 限定でなく任意層に一般化されたことが MUST との対称性で正当化される。 |
| §4 ⇄ §5.3 Foundation Example | **整合**。旧 Example の `font: inherit` 行が削除され、注記が §5.2 への移送理由を述べる。§5.2 Reset 本文（L302-317）は「ブラウザデフォルトの初期化」が責任と定義しており、`font: inherit` がそこに帰属することと矛盾しない。 |
| §4 MAY ⇄ Requirements Index MAY 節 | **整合**。新規 `\| §4 \| MAY [プラットフォーム不変条件の保護] \| §4 要求レベル \|` 行を § 昇順で先頭に挿入済（§5.1 行の直前）。 |
| §4 MUST NOT [!important の使用制限] ⇄ Requirements Index MUST NOT 節 | **整合**。L832 既存行に変更なし。文言修正（「ただし…」の追加）は要求レベル本質を変えず、Index 1 行サマリは依然正確。 |
| §2 准拠条件 ⇄ §4 MUST NOT 文言 | **整合**。§2 L84「全 MUST / MUST NOT ルールに違反しないこと」は維持。§4 の MUST NOT が「例外」で限定された場合の準拠判定は例外節の具体列挙に従えば一意に決まる（type 2 と type 3 はそれぞれ MUST / MAY の参照で内部完結）。 |
| dangling reference | `grep -E "例外: Reset 層\|Reset 層の内部実装\|優先度逆転"` で 0 件。旧見出し文字列への参照は spec.md 内に残存していない。 |

### 3. 列挙閉性（Cat3 が `[hidden]` のみに閉じ無制限化していないか）

- MAY [プラットフォーム不変条件の保護] 本文: 「許容される対象は `[hidden]` 属性（`until-found` 値を除く）に対する `display: none` に限る。本仕様が列挙しない対象への拡張は MUST NOT [!important の使用制限] に従う。」
- 評価: **閉列挙が明文化されている**。「本仕様が列挙しない対象への拡張は MUST NOT に従う」という規範文により、`[hidden][display: none]` 以外の任意のプロパティ / セレクタを「不変条件保護」の名目で `!important` 化することは禁止される。
- 将来の拡張は spec の改訂（minor → MAY 行追加 / major → MUST 例外拡張）で対応する設計になっている。

### 4. Gap B: Reset canonical 判断と §5.2 Reset Test 整合

- §5.2 Reset Test（L308）: 「このスタイルはブラウザデフォルトの初期化か？」 → Yes → Reset。
- フォーム要素の `font: inherit` は、ブラウザがフォーム要素に与える既定フォント（`-webkit-small-control` 等のシステム既定）を親要素のフォントに置き換える初期化に該当。すなわち「ブラウザ間差異の吸収」かつ「ブラウザデフォルトの初期化」に合致。Reset Test に Yes。
- §5.3 Foundation Test（L343）: 「このスタイルは、全ページ共通の基本スタイルか？」 → `font: inherit` は **基本スタイルの付与ではなく初期化** であり Foundation Test の意図とずれる。
- 評価: **§5.2 Reset 帰属が canonical** で正しい。新 §5.3 注記の説明文は Reset Test 定義と整合。
- 実装側証跡: `~/Git/mflocss-starter/src/assets/css/reset/reset.css` L86-92 は `:where(button, input, select, textarea) { font: inherit; ... }` を Reset ディレクトリ + `@layer` reset として実装済。spec 改訂は starter 実装と一致する方向の修正。

### 5. バージョニング

- 本変更は `## [v1.0] - Unreleased` 内の追記であり、版番号変更は不要。CHANGELOG.md の `### Changed` 節への追記のみ。
- **release 後の同等変更を行う場合の注記**: spec §2 バージョニング（L94）は「MUST/MUST NOT の追加・削除・変更（他レベルからの昇格・降格を含む）」を **メジャー** と規定している。本 PR で行った MUST NOT [!important の使用制限] への「ただし『例外: `!important` の許容範囲』に定める場合を除く」の追加は、例外範囲の **拡張**（Reset 限定 → 任意層）であり実質的に MUST NOT の制約緩和に該当する。release 後にこれと同等の変更を行う場合は **メジャー版（v2.0.0）相当** の扱いとなる。本 PR は Unreleased 内なので影響なし。
- 新規 MAY [プラットフォーム不変条件の保護] の追加は単独では minor 相当（§2 L95「SHOULD/MAY の追加…」）だが、上記 MUST NOT 例外拡張とセットで release 後に行う場合はメジャー側に吸収される。

## 結論まとめ

5 評価軸すべてで PASS。修正提案なし。

cortex#1214 が指摘した 3 課題（Gap A / Gap B / Gap C）は本 PR で構造的に解消され、`!important` の許容範囲が「抽象原則 + 閉列挙された 3 類型」として体系化された。starter / mflocss.dev 実装の既存事実（`[hidden]!important` + form `font:inherit`(Reset)）が spec で sanction され、spec ↔ 実装の整合が回復する。

## Codex クロスレビュー対応

PR #106 作成後の codex review で 2 件の P2 指摘を受領。判定は以下:

- **P2-1（Informative 注記の純度 — 抽象原則の Normative 化 / @scope 将来注記の Normative 化）= 却下**
  理由: cortex#1214 が「抽象原則（純粋な追加スタイル）+ カテゴリ列挙（プラットフォーム不変条件 / Reset 完全な reset）の二段構造」と「@scope 移行で将来不要になる可能性も Informative として残す」ことを明示的に **設計要件** として要求している。抽象原則の Informative 配置は本再設計の中核成果物（「閉列挙だけでは将来 case に追従不能 → 抽象原則を判断基準として与える」という Issue 設計意図）であり、Normative 純度ガイドラインより Issue の設計意図が支配。@scope 注記も将来の MAY 削除可能性を示す informative 注記として spec 純度的にも妥当。
- **P2-2（Gap B Example 裏付け — §5.2 Reset Example に form `font:inherit` を追加）= 採用**
  理由: 「Reset が canonical」を本文・注記で確定する以上、§5.2 Reset Example にもその実例が示されているべき。注記のみの裏付けでは「§5.2 を読んで Reset 層の代表的構成を把握する」読者が form 規則の所属を判定できない（spec 整合性ギャップ）。starter reference 実装 `~/Git/mflocss-starter/src/assets/css/reset/reset.css` L86-92 とも一致する方向の追加。§5.3 Foundation Example から form 規則を削除した本 PR の改訂と対をなす。

**反映内容**: §5.2 Reset Example 内（`@layer reset { ... }` ブロック末尾）に以下を追加。

```css
:where(button, input, select, textarea) {
  font: inherit;
}
```

§5.3 の Informative 注記（「フォーム要素のフォント継承は Reset 層に配置」）は境界説明として有用なので維持。§5.3 Example は変更なし。

**codex 集計**: P1=0 / P2=2（1 採用 / 1 却下）/ P3=0。本対応により Example と注記が一致し、§5.2 単独でも Reset canonical の事実が読み取れる構造になる。
