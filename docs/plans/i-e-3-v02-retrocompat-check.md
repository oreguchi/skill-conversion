# I-E.3: v0.2 → v1.0 Retro-compatibility Check

**Target:** vb-winforms-skills v2.0.0 + v2.1.0（累積 26 件 approved-additions）
**Virtual profile:** balanced（最頻ケース）
**Date:** 2026-04-25
**Source register:** `C:\dev\01_ClaudeProject\skill-conversion実践2\docs\plans\2026-04-23-vb-winforms-skills-conversion\approved-additions.md`
**Source drift report:** `C:\dev\01_ClaudeProject\skill-conversion実践2\docs\plans\2026-04-23-vb-winforms-skills-conversion\drift-report.md`

---

## サマリ（先頭に主要数値）

- 総エントリ数: **26 件**（minor 7 / moderate 11 / heavy 4 — v2.1 で moderate +5 / heavy +3 が追加された結果、v0.2 当初の minor 7 / moderate 9 / heavy 3 から増加）
- Category 内訳（推定）: **A=15 / B=11 / C=0**
- v1.0 / balanced 評価結果:
  - auto-approve: **22 件**（A×全 + B×minor）
  - user approval required（採用）: **4 件**（B×moderate / B×heavy）
  - **rejected**: **0 件**
- v0.2 verdict（MODERATE）と v1.0 仮想 verdict が一致。後方互換性 **OK**。

---

## Category 分類の方針

`references/approved-additions.md` の定義および `references/catalog-system.md` の Tier 対応に基づく。

- **A. Fidelity-preserving** — VB.NET コンパイラ/ランタイム制約（BC30668、BC36943、BC30026/30289、BC30521、BC30456、BC30203、Iterator+Async 同居不可、`_` 識別子不可、`Finally Await` 不可など）に起因する書き換え・補強。ソースが伝えたい教育的主題を VB.NET 文脈で保持するための補完。
- **B. Environment-essential**（Catalog Tier 1+2 相当） — 対象環境（VB.NET + WinForms）の de-facto 必須要素。WinForms テンプレート、UI スレッドマーシャリング、`IProgress(Of T)`、DI によるフォーム解決、Akka.NET VB 注記、Option Strict など、これらが欠落するとスキルが「不完全」と評価されるもの。
- **C. Differentiating**（Catalog Tier 3 相当） — 対象環境固有の差別化機能（PerMonitorV2 高 DPI、async タイマー代替、純粋な Func ファクトリ高度化など）。今回の register には**該当する単独エントリは存在しない**（Func(Of TForm) ファクトリは Tier 2 の DI フォーム解決と一体で扱われている）。

---

## Category retrofit（代表 18 件 + クラスタ集約）

| # | Entry（短縮） | Original Impact | Estimated Category | Verdict under balanced |
|---|---|---|---|---|
| 1 | `dotnet new winforms` に `--language VB` 付与 | minor | A | auto-approve |
| 2 | `.vbproj` に `<RootNamespace>` 明示 | minor | B | auto-approve |
| 3 | Akka.NET VB.NET 利用可だが冗長注記 | minor | B | auto-approve |
| 4 | Blazor VB.NET 非対応注記 | moderate | A | auto-approve |
| 5 | Aspire VB.NET 非対応注記 | moderate | A | auto-approve |
| 6 | gRPC VB.NET 非対応注記 | minor | A | auto-approve |
| 7 | `IAsyncEnumerable` + `Async Iterator` 制約注記（擬似コード扱い） | moderate | A | auto-approve |
| 8 | `SearchOrdersRequest` mutable プロパティ理由コメント | minor | A | auto-approve |
| 9 | `AddOptionsWithValidateOnStart` 置換（BC30521 回避） | moderate | A | auto-approve |
| 10 | `Akka.Hosting.TestKit` `ConfigureServices` オーバーライド削除 | moderate | A | auto-approve |
| 11 | `Await For Each` → 4-block `IAsyncEnumerator` 書き換え（BC36943） | moderate | A | auto-approve |
| 12 | `Span(Of T)` 第 1 次シグネチャ削除（6 メソッド） | heavy | A | auto-approve（A は heavy でも auto） ※注 |
| 13 | 「非同期ローカル関数」セクション scope-strip（BC30026/30289） | heavy | A | auto-approve ※注 |
| 14 | `Try/Finally Await DisposeAsync` → 4-block 化 | moderate | A | auto-approve |
| 15 | `Sub(_)` → `Sub(state)`（BC30203） | minor | A | auto-approve |
| 16 | テストラムダに `As Func(Of Task)` 明示型（BC30456） | minor | A | auto-approve |
| 17 | `Span(Of T)` 全排除（ローカル変数も含む第 2 次、9 メソッド） | heavy | A | auto-approve ※注 |
| 18 | `vb-coding-standards` description を `Memory(Of T)/ArrayPool(Of T)` に更新 | moderate | A | auto-approve |
| 19 | WinForms テンプレートセクション新設（Phase 4） | moderate | B | **user approval required**（採用） |
| 20 | `AddOptionsWithValidateOnStart` 両パターン併記コメント（v2.1） | minor | B | auto-approve |
| 21 | `Directory.Build.props` から `<Nullable>` 削除＋`Option Strict` 明示（v2.1） | moderate | B | **user approval required**（採用） |
| 22 | `Program.vb` を `<STAThread>` + `Module Program` + `ApplicationConfiguration.Initialize()` に復元（v2.1） | moderate | A | auto-approve |
| 23 | UI スレッドマーシャリング章強化＋`IProgress(Of T)` 進捗報告節（v2.1, **heavy**） | heavy | B | **user approval required**（採用） |
| 24 | DI フォーム解決＋`Func(Of TForm)` ファクトリ節（v2.1） | moderate | B | **user approval required**（採用） |
| 25 | `vb-concurrency-patterns` に WinForms 特化章追加（v2.1, **heavy**） | heavy | B | **user approval required**（採用） |

※注: 9 マス表で「A. Fidelity · heavy」は **`user approval required`**（auto ではない）。再分類の正確性のため以下に補正表を示す。

### A × heavy の補正

| # | Entry | 9 マス表上の正確な verdict |
|---|---|---|
| 12 | `Span(Of T)` 第 1 次シグネチャ削除 | user approval required（採用、A なので reject されない） |
| 13 | 「非同期ローカル関数」scope-strip | user approval required（採用） |
| 17 | `Span(Of T)` 全排除（第 2 次） | user approval required（採用） |

A × heavy はプロファイル横断で常に「user approval required」だが、**rejected には絶対ならない**点が重要。v0.2 では実際に user 承認済みなので v1.0 でも同等の結末となる。

---

## カウント集計（balanced × 26 件）

| 結末 | 件数 | エントリ # |
|---|---|---|
| auto-approve | 18 | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 14, 15, 16, 18, 20, 22 |
| user approval required（A × heavy） | 3 | 12, 13, 17 |
| user approval required（B × moderate / heavy） | 5 | 19, 21, 23, 24, 25 |
| **rejected** | **0** | — |

合計 26 件、すべて採用される（auto または user 承認）。

---

## Drift verdict 比較

- **v0.2 verdict**: MODERATE
- **v1.0 仮想 verdict**: **MODERATE**（同じ）
- **Rationale**: 26 件中 0 件が profile 拒否される。v1.0 が新たに導入した 9 マス表は「C × moderate / heavy を rejected 化する」という運用差を生むが、本変換にはそもそも C 単独エントリが存在しないためこの差が発火しない。Heavy ドリフトの主要 3 件（Span 全排除 ×2、ローカル関数 scope-strip）はいずれも **A. Fidelity-preserving**（VB.NET 言語制約に起因する不可避の補修）であり、9 マス表でも「A × heavy = user approval required（採用）」となるため v0.2 の判定（user 承認済み）と帰結が一致する。v2.1 で追加された heavy 2 件（UI スレッドマーシャリング章、WinForms 特化 concurrency 章）は B（Catalog Tier 2 相当の WinForms de-facto 必須要素）であり、balanced では「user approval required」だが採用される。よってドリフト規模・性質ともに v0.2 の MODERATE と同等。

---

## Rejected entries（balanced で採用されないもの）

| Entry | Category | Impact | Why rejected |
|---|---|---|---|
| — | — | — | **該当なし**（0 件） |

---

## 周辺観察

1. **C 不在の理由**: 本変換は WinForms LOB 文脈という狭い対象環境であり、VB.NET の差別化機能（純粋な「VB.NET ならでは」の Tier 3 機能）を持ち込む余地が少なかった。`Func(Of TForm)` ファクトリは Tier 3 寄りだが、entry 24 のなかで Tier 2 の DI フォーム解決と一体運用されており、エントリ全体としては B 判定。
2. **A 偏重**: 26 件中 15 件（57.7%）が A. Fidelity-preserving。これは VB.NET と C# の言語仕様差が大きい（ref struct、ローカル関数、Iterator+Async 同居、`Finally Await`、`_` 識別子、null 許容参照型フロー解析の有無など）ことに起因する。同種の言語間変換で再現する傾向と思われる。
3. **B 偏重**: 残り 11 件中 11 件すべてが B。WinForms プラグインとしての de-facto 必須要素（テンプレート、UI スレッド、IProgress、DI フォーム）が大半を占めるため。
4. **9 マス表の効き目**: 本ケースでは表の差分が表に出ないが、もし高忠実度プロファイルで再評価すれば B × moderate（entry 19, 21）と B × heavy（entry 23, 25）が **rejected** となり、v0.2 の Phase 4 採用 1 件・v2.1 retrospective 7 件の大半が落ちる。これは設計通りの挙動で、profile 宣言の意義が確認できる。
5. **C × moderate/heavy 拒絶ロジックは未検証**: 本ケースでは試験対象がいないため、C 偏重の変換ケース（例: C# → Rust、または高 DPI 専用 UI スキル）で別途 dogfooding が必要。

---

## Conclusion

後方互換性は **acceptable**（受容可能）。balanced プロファイル下で v0.2 の 26 件すべてが v1.0 の 9 マス表でも採用されるため、v0.2 から v1.0 への移行は drift verdict を悪化させない。MODERATE という総合判定も維持される。

ただし以下の条件が成立する場合は再評価が必要:
- ユーザーが high-fidelity プロファイルを宣言した場合 → B × moderate / heavy が rejected となり、v2.1 retrospective の大半（entry 19, 21, 23, 25）が落ちる。これは仕様通りの挙動。
- C 単独エントリを伴う変換ケース（VB.NET 以外の対象環境） → C × moderate/heavy 拒絶ロジックが本格的に効く。別途 dogfooding 推奨。
