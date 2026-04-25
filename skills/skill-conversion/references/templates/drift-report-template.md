# Drift Report: <target-skill-name>

**Source:** <source-skill-path>, version <N>
**Target:** <target-path>
**Conversion dimensions:** <list>
**Conversion Profile:** high-fidelity / balanced / high-utility   (v1.0 NEW)
**Phase 4 opt-in state:** in / out (out is allowed only when profile = high-fidelity; reason recorded in plan-doc §7.1)
**Report date:** <YYYY-MM-DD>

## Catalogs used (v1.0 NEW)

| Catalog | Path | Persistence status | Last updated | Tier reference range |
|---|---|---|---|---|
| <name> | references/catalogs/<file>.md | persistent / memory-only | YYYY-MM-DD | Tier 1 / Tier 1+2 / Tier 1+2+3 |

## Frontmatter verification

| Skill | Field | Source value | Target value | Status |
|---|---|---|---|---|
| <name> | name | ... | ... | ✓ / ✗ |
| <name> | description | ... | ... | ✓ / ✗ |
| <name> | invocable | ... | ... | ✓ / ✗ |
| <name> | compatibility | ... | ... | ✓ / ✗ |

(Every ✗ must link to an approved-additions entry. Unregistered ✗ blocks sign-off.)

## Phase 5 status

| Target skill | Status | Evidence |
|---|---|---|
| <skill> | PASS / SKIPPED / USER-PENDING | `phase5-result.md` / source frontmatter quote / `phase5-handoff.md` |

## Summary

- Total approved additions: N
  - Phase 1: X (minor: Xm, moderate: Xmo, heavy: Xh)
  - Phase 4: Y (minor: Ym, moderate: Ymo, heavy: Yh)  [or "SKIPPED by user choice"]
- Total rejected proposals: Z [Phase 4 only]
- Overall drift verdict: LOW / MODERATE / HIGH
- Rationale for verdict: <one paragraph>

## Phase 1 approved additions

| Added-on | Location | Summary | Category | Reason | Impact | Approver |
|---|---|---|---|---|---|---|

## Phase 4 approved additions

(If Phase 4 was SKIPPED, state so and omit the table.)

| Added-on | Location | Summary | Category | Reason | Impact | Approver |
|---|---|---|---|---|---|---|

## Phase 3 adopted-API roster (v1.0 NEW)

| API | Officiality | Package | Both-pattern annotation | Source URL |
|---|---|---|---|---|
| <api> | Official / Standard / Peripheral / Unknown | <pkg> | yes / no / N/A | <url> |

## Profile-rejected proposals (v1.0 NEW)

Additions that the profile mechanically rejected (and are therefore NOT adopted).

| Date | Location | Proposal summary | Category | Impact | Rejection reason |
|---|---|---|---|---|---|
| YYYY-MM-DD | <loc> | <summary> | A / B / C | minor/moderate/heavy | profile=<X> rejects category <Y> at this impact |

## Phase 4 rejected proposals

(If Phase 4 was SKIPPED, omit this section.)

| Date | Location | Proposal summary | Rejection reason |
|---|---|---|---|

## Drift verdict detail

<One paragraph of human judgment describing the qualitative drift.>

## User confirmation

- [ ] User reviewed and accepted drift
- Reviewer: <user identifier>
- Date: <YYYY-MM-DD>
