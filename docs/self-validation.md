# Self-validation evidence — skill-conversion v0.1.0

This plugin was run through its own Phase 4 (empirical-prompt-tuning) as part of its release criteria. This document summarizes the evidence.

## Method

- 3 scenarios × 3 iterations = 9 blind-evaluator runs
- Each run: a fresh subagent reads only `SKILL.md` (references loaded only when SKILL.md directs), performs the Phase 0 plan, reports findings under a fixed schema (成果物 / 要件達成 / 不明瞭点 / 裁量補完 / 再試行)
- User-gated individual approval on every proposed change to SKILL.md

## Scenarios

| ID | Source skill | Target | Dimensions exercised |
|---|---|---|---|
| S1 | `superpowers:systematic-debugging` | Go variant | Technical only (Python → Go) |
| S2 | `superpowers:writing-plans` | Japanese variant | Locale only (EN → JA) |
| S3 | `superpowers:test-driven-development` | JA Copilot CLI + Go | All three dimensions |

## Iteration record

| Iteration | Scenarios | [critical] pass rate | Retries | Triggering signals (≥2/3) | Avg duration | Outcome |
|---|---|---|---|---|---|---|
| 0 (static) | alignment check | — | — | — | — | PASS (no drift) |
| 1 | S1, S2, S3 | 100% | 0/0/0 | 2 (F1 plan-doc subdir, F2 §8 Agent=identity rule) | 164s | user-approved → apply |
| 2 | S1, S2, S3 | 100% | 0/0/0 | 1 (F4 source-skill layout patterns) | 192s | user-approved → apply |
| 3 | S1, S2, S3 | 100% | 0/0/0 | **0** | 221s | **pragmatic convergence** |

## Approved additions

- **F3** (Iter1) — Target skill naming convention: kebab-case, `<source-name>-<marker>` form, with examples. **Impact: minor.**
- **F4** (Iter2) — Source-skill layout enumeration: standard / flat / `@`-link / single-file patterns, with handling per pattern. **Impact: moderate.**

Total: 2 entries, 0 heavy, 0 rejected.

## Clarifications (not registered)

Per SKILL.md §Phase 4 rule, "clarification only (impact: minor) → treat as Phase 2 change, no register entry." Applied in Iter1:

- F1 — Plan-doc subdirectory form permitted for iterative testing
- F2 — §8 "N/A — Agent dimension = identity" rule made explicit
- U3 — "references" narrowed to `references/` subdirectory; related-skills links excluded
- U4 — `conversion-dimensions.md` declared mandatory reading for item 3
- U5 — Evaluation-scenario skeleton granularity specified
- U6 — Explicit reading policy for items 3/5/7/8
- U9 — "high-stakes skill" criterion defined (3 conditions)

## Known minor items (logged, not pursued in v0.1.0)

Found at Iter3 with signal 1/3 and judged not worth another iteration:

- **W1** — `version` field handling when source SKILL.md lacks frontmatter version
- **W2** — Flat-layout treatment of non-`.md` companion files (`.ts`, `.sh`, etc.)

Both are candidates for a minor clarifying commit in a future v0.2.

## Final drift verdict

**LOW.** All Phase 4 changes concentrated in §Phase 0 item 1 plus one Phase 2 definition addition. Process skeleton (Phase 0→5 flow, user-gate discipline, audit-trail principle) unchanged across all iterations.

User-confirmed 2026-04-20 by oreguchi.

## Artifacts (development-time)

The full evidence trail lived in the development repo at `docs/plans/2026-04-20-skill-conversion/`:

- `eval-scenarios.md` — scenario definitions
- `empirical-results.md` — per-iteration metrics table
- `approved-additions.md` — register
- `drift-report.md` — final drift summary
- `iter{1,2,3}-s{1,2,3}-phase0-plan.md` — 9 blind-executor output plans

These are not redistributed with the plugin (development-internal), but the summary above is faithful to them.
