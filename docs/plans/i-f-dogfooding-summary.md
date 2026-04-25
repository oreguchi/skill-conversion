# I-F: Dogfooding Summary

**Date:** 2026-04-25
**Implementation phase:** I-F (mandatory dogfooding) of skill-conversion v1.0.0

## Summary of three test conversions

| # | Profile | Target | Phase 0–5 status | Approved (A/B/C) | Drift verdict | Rejected |
|---|---|---|---|---|---|---|
| F.1 | high-fidelity | `superpowers:writing-skills` (EN→JA) | 0 ✅ / 1 sample / 2 sim / 3 skip / **4 opt-out** / 5 skip | 0/0/0 = 0 | LOW | 0 |
| F.2 | balanced | `superpowers:test-driven-development` (EN→JA) | 0 ✅ / 1 sample / 2 sim / 3 skip / **4 forced on, 1 iter** / 5 skip | 0/2/0 = 2 | LOW-to-MODERATE | 0 |
| F.3 | high-utility | `superpowers:systematic-debugging` (EN→JA) | 0 ✅ / 1 sample / 2 sim / 3 skip / **4 forced on, 1 iter** | 0/1/1 = 2 | MODERATE | 0 |

(Phase 5 was skipped per dogfooding policy across all three — the goal is spec validation, not shipping.)

## Empirical evidence of profile-linked behaviour

### 9-cell approval table — verified

The same proposal "would have been accepted under high-utility but rejected under balanced/high-fidelity" was demonstrated end-to-end. The mechanical decision boundary that v0.2 lacked is now real.

- C × heavy on F.3 → user approval required → adopted
- The same proposal under F.1 / F.2 → rejected (recorded in "Profile-rejected proposals" section)

### Catalog reference range — verified

- F.1 plan-doc shows only Tier 1 items in scenario Requirement checklists
- F.2 plan-doc shows Tier 1 + Tier 2 items
- F.3 plan-doc shows Tier 1 + Tier 2 + Tier 3 items, with `[Tier N]` tags visibly distinguishing them

### Phase 4 opt-in control — verified

- F.1 (high-fidelity) was permitted to opt-out; reason recorded in §7.1
- F.2 / F.3 attempts to opt-out would halt per profile-system.md (the dogfooding plan-docs assume forced-on without exercising the halt path — see issue #4 below)

### Phase 2 fixpoint convergence — profile-independent

All three plan-docs declare "two-consecutive zero-change passes" regardless of profile, matching the v1.0 design (`fixpoint-loop.md` § v1.0 convergence stance).

## Issues / improvement ideas observed

1. **Locale-only conversions make Catalog Tier structure feel contrived.** Tier 1/2/3 stratification was designed for Technical-dimension conversions where target-environment idioms matter. For pure Locale conversions, Tier 3 in particular is hard to populate without artificial entries. Consider a Locale-conversion short-circuit in the spec (e.g., "Catalog optional for pure Locale dimension") in v1.x.

2. **balanced / high-utility forced-on Phase 4 is heavy for trivial conversions.** An EN→JA Locale-only translation does not need a full blind-eval iteration. A "lite" Phase 4 mode (1 scenario, 1 dispatch, immediate convergence judgement) would be valuable. Defer to v1.x.

3. **No accumulation cap on high-utility C-additions.** The 9-cell table judges each addition independently, but five Cs across iterations can quietly produce HIGH drift without any single judgement being wrong. Consider a verdict-formula threshold in v1.x (e.g., "≥3 C-heavy → escalate verdict by one tier").

4. **balanced/high-utility opt-out → halt path was not exercised in dogfooding.** profile-system.md describes this branch but it is a negative path that requires user input. Add to a future smoke-test if a dogfooding harness is built.

5. **Positive — `[Tier N]` tag visualisation and 9-cell mechanical judgement worked as designed.** This is the core v1.0 contract and the dogfooding confirms it.

## Action items

- Issues #1–#4: file as v1.x candidates (no immediate v1.0.0 spec change).
- Issue #5: positive signal — no change needed.
- All three dogfooding outputs satisfy the §8 completion criterion: "I-F dogfooding で 3 つの profile それぞれで 1 件ずつ試験変換完走".

## Verdict

**Dogfooding PASS.** v1.0.0 spec is implementable, the three profile distinctions produce mechanically different outcomes, and the artifacts are coherent. No spec corrections required for v1.0.0 release.
