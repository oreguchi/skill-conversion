# Dogfooding Result — high-utility profile

**Target skill:** `systematic-debugging` → `systematic-debugging-ja` (`C:\Users\workm\.claude\plugins\cache\claude-plugins-official\superpowers\5.0.7\skills\systematic-debugging\SKILL.md`)
**Profile:** high-utility (高有用性)
**Date:** 2026-04-25

> 当初依頼の `superpowers:debugging` は superpowers 5.0.7 のキャッシュに不在だったため `systematic-debugging` を採用。F.1 (`writing-skills`) / F.2 (`test-driven-development`) と異なる target である。

## Phase completion summary

| Phase | Status |
|---|---|
| Phase 0 | COMPLETE |
| Phase 1 | SAMPLE (3 抜粋: SKILL.md 冒頭 / Phase 4 由来 Tier 3 追加 / Common Rationalizations 表 B × minor 追加) |
| Phase 2 | SIMULATED (single-pass) |
| Phase 3 | SKIPPED (no buildable code blocks) |
| Phase 4 | **FORCED ON, 1 iteration simulated, Tier 3 / C × heavy 追加 1 件採用** |
| Phase 5 | SKIPPED per dogfooding policy |

## Approved-additions counts

| Category | minor | moderate | heavy |
|---|---|---|---|
| A | 0 | 0 | 0 |
| B | 1 | 0 | 0 |
| C | 0 | 0 | **1** |

Total approved: **2**
Total rejected (9-cell): 0（high-utility は構造的に rejected セルが少ない）

## Drift verdict

**MODERATE.** Phase 4 由来の **Tier 3 / C × heavy** が 1 件 register に入った。Profile が許容する utility 拡張範囲だが、heavy 1 件の重みで MODERATE と判定。

## v1.0 mechanism observations

- **Profile linkage:** high-utility 宣言 → 4 軸全てが許容方向に開く（Phase 4 forced on / 9-cell が permissive 帯 / Catalog Tier 3 まで visible）。`[Tier 3]` タグが scenario に登場するのは high-utility のときだけで、profile 経路の差別化点が drift-report 上から目視確認できる。
- **Catalog reference scope:** Tier 3 が **実際に Phase 4 の Category 判定で決め手になった**（提案を「Catalog Tier 3 該当 → Category C」と分類するロジックが機能）。3 つのプロファイル中で Tier 3 が実発火するのは high-utility のみで、機構の差別化がはっきり観察できる。
- **9-cell judgement:** 本 dogfooding の中核観察。**high-utility 列は C × heavy が user approval で採用可** という点が決定的に balanced と異なる。実発火 1 件で承認フォームを通り採用。balanced だと同じ提案は **rejected (default)** に落ちて register に入らないはずで、profile 切替で結果が機械的に変わる挙動を確認できた。
- **Phase 3 source-confirmation:** Phase 3 SKIPPED (Locale-only) のため adopted-API roster は空。Locale 変換では本機構の出番が乏しいことが 3 プロファイル共通で確認された。

## Issues / improvement ideas for skill-conversion

- **Issue 1: high-utility の累積 drift 上限が spec にない。** 本 dogfooding では Phase 4 を 1 iteration しか回していないが、複数 iteration 回すと C × heavy が累積する余地がある。9-cell は単発判定の仕組みで、累積に対する制限はない。**改善案：** drift-report 「Summary」に閾値（例: C × heavy が 3 件超なら自動的に drift verdict = HIGH に強制）を導入するか、Phase 4 convergence 基準に「新規 C × heavy 追加 = 1 件以下」を追加するか。

- **Issue 2: high-utility の Tier 3 採用判定が subjective に流れやすい。** Catalog の Tier 3 セクション内容が generation-subagent の出力依存で、本 dogfooding でも手書きの「Claude Code 環境固有の慣用表現」「JA 圏文化に固有の作法」と曖昧。Tier 3 の境界が profile 機構の決め手なのに、その境界の明確化が generation-subagent-prompt.md に強く依存する。**改善案：** Tier 3 判定の self-check 質問（例: 「この能力は target reader の 50% 以上が即座に使うか？ Yes なら Tier 2 へ降格」）を catalog-system.md に追記する。

- **Issue 3 (positive): プロファイル切替で結果が機械的に変わることを 3 ケース横断で確認できた。** 同じ Tier 3 / C × heavy 提案を仮想的に F.1 (high-fidelity) / F.2 (balanced) / F.3 (high-utility) に当てはめると：
  - high-fidelity: rejected (採用なし)
  - balanced: rejected (採用なし)
  - high-utility: user approval → 採用
  という分岐が 9-cell 表どおりに機能。profile 宣言が drift 結果を mechanically 決めることが可視化された。これは v1.0 の中核設計目標が達成されている証拠。

- **Issue 4: dogfooding スコープでは Tier 3 が「Locale 変換でも実体化できる」ことを示したが、Tier 3 は本来 Technical 変換時に最も価値がある。** Locale-only で Tier 3 を作り込むのは contrived な側面があり、本物の運用観察は Technical 変換 (例: Python → Go) で改めて行うべき。
