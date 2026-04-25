# Drift Report: test-driven-development-ja

**Source:** `C:\Users\workm\.claude\plugins\cache\claude-plugins-official\superpowers\5.0.7\skills\test-driven-development\SKILL.md`, version superpowers 5.0.7
**Target:** `test-driven-development-ja`（dogfooding 仮想出力）
**Conversion dimensions:** Locale (en-US → ja-JP)
**Conversion Profile:** balanced (バランス)   (v1.0)
**Phase 4 opt-in state:** **in** (forced on by balanced profile, opt-out attempt は仕様上 halt するため最初から in 固定)
**Report date:** 2026-04-25

> **Dogfooding 注記。** 本変換は v1.0 spec 検証のための dogfooding。Phase 2 simulated single-pass、Phase 3 SKIPPED (no buildable code)、Phase 4 1 iteration を simulate して 1 件の addition を採用、Phase 5 SKIPPED per dogfooding policy。

## Catalogs used (v1.0 NEW)

| Catalog | Path | Persistence status | Last updated | Tier reference range |
|---|---|---|---|---|
| Locale-JA (memory only) | (memory only) | memory-only | 2026-04-25 | **Tier 1 + Tier 2** |

Tier 1 + Tier 2 が Phase 0 §6 と Phase 1 翻訳ガイドに参照された。Tier 3 は **意図的に未参照**（balanced プロファイルの設計どおり）。

## Frontmatter verification

| Skill | Field | Source value | Target value | Status |
|---|---|---|---|---|
| test-driven-development-ja | name | `test-driven-development` | `test-driven-development-ja` | ✓ |
| test-driven-development-ja | description | `Use when implementing any feature or bugfix, before writing implementation code` | (同一) | ✓ |
| test-driven-development-ja | invocable | (省略 = true) | (省略 = true) | ✓ |

## Phase 5 status

| Target skill | Status | Evidence |
|---|---|---|
| test-driven-development-ja | **SKIPPED (dogfooding policy)** | dogfooding scope notice in `00-master-plan.md` |

## Summary

- Total approved additions: **2**
  - Phase 1: 1 (minor: 1, moderate: 0, heavy: 0)
  - Phase 4: 1 (minor: 0, moderate: 1, heavy: 0)
- Total rejected proposals: **0** (Phase 4 1 iteration では reject なし。9-cell rejected プールにも実発火なし)
- Profile-rejected proposals (9-cell mechanically rejected): 0 件 (実発火) / 仮想 2 件は phase1-sample.md の 9-cell 例示で言及
- Overall drift verdict: **LOW-to-MODERATE**
- Rationale for verdict: Locale 変換ながら balanced プロファイル下で B 系の小規模追加が 2 件 register に入った（pytest 言い換え 1 件 + Phase 4 由来のロケール注記 1 件）。原作者意図への drift は限定的で、JA 圏読者の utility 改善に資する範囲に収まっている。heavy も C 系も 0 件のため、profile が想定する drift の上限（balanced は B-moderate / C-minor までを許容）を超えていない。

## Phase 1 approved additions

| Added-on | Location | Summary | Category | Reason | Impact | Approver |
|---|---|---|---|---|---|---|
| 2026-04-25 | SKILL.md §Verify RED 直前 | pytest / npm test 等の JA 圏慣用テスト実行コマンドを各言語に置き換えて読む旨の一行注記 | B | JA 圏 TDD ユーザはテストランナーを母国語混じりで呼び出すのが慣例。Locale-JA Tier 2 に該当。 | minor | oreguchi |

## Phase 4 approved additions

| Added-on | Location | Summary | Category | Reason | Impact | Approver |
|---|---|---|---|---|---|---|
| 2026-04-25 | SKILL.md §Verify RED (Watch It Fail) 末尾 | 日本語ロケール環境での pytest 出力解釈の補足（LANG=C.UTF-8 / pytest -v 推奨） | B | Blind subagent (simulated) が unclear-point として報告。balanced × B × moderate → user approval required で承認。 | moderate | oreguchi |

## Phase 3 adopted-API roster (v1.0 NEW)

| API | Officiality | Package | Both-pattern annotation | Source URL |
|---|---|---|---|---|
| (none — Phase 3 SKIPPED, no buildable code blocks) |  |  |  |  |

## Profile-rejected proposals (v1.0 NEW)

実発火 0 件。`phase1-sample.md` 内の 9-cell 例示で「日本企業の TDD 導入事例（C × heavy）」「Sinon 高度パターン（C × moderate）」を仮想ケースとして挙げ、これらは balanced 列で **rejected** に落ちることを確認。実 dogfooding 中はこれら C × moderate/heavy の提案を出さなかったため空。

| Date | Location | Proposal summary | Category | Impact | Rejection reason |
|---|---|---|---|---|---|
| (none — no actual proposals during dogfooding) |  |  |  |  |  |

## Phase 4 rejected proposals

| Date | Location | Proposal summary | Rejection reason |
|---|---|---|---|
| (none — Phase 4 1 iteration shipped 1 addition, no rejections) |  |  |  |

## Drift verdict detail

**判定: LOW-to-MODERATE。** balanced プロファイルが想定する utility 拡張の帯（B × minor auto-approve + B × moderate user approval）にちょうど収まる 2 件の追加で完了。C 系・heavy は 0 件で、profile が要求する drift 上限を逸脱していない。

dogfooding 上の留保：Phase 4 を 1 iteration しか simulate していないため、実運用で複数 iteration 走らせた場合に B 系追加が累積するリスクは別途観察が必要。

## v1.0 4 軸の挙動チェック

| 軸 | balanced の規定値 | 本 dogfooding での実観測 |
|---|---|---|
| Phase 4 opt-in state | forced on | opt-out 試行は規約上 halt → 試行せず最初から forced on のまま実行 |
| approved-additions threshold | standard (A + B) | A 系発火なし、B 系 2 件 (minor 1 + moderate 1)、C 系 0 件 |
| Catalog reference range | Gotchas + Tier 1 + Tier 2 | **Tier 1 + Tier 2** が scenario チェックリストに `[Tier 1]` `[Tier 2]` で embed され可視化 |
| Phase 2 fixpoint convergence | high-stakes | high-stakes 扱いで simulated（本来 2 連続ゼロ変更要件） |

## User confirmation

- [ ] User reviewed and accepted drift
- Reviewer: oreguchi
- Date: 2026-04-25 (dogfooding scope)
