# Drift Report

## Purpose

At completion, summarize **every deviation from the source skill**. This makes the converted skill's provenance auditable and surfaces when a conversion drifted further than intended.

## When to produce

After `superpowers:verification-before-completion` confirms completion criteria 1-4, produce the drift report. User-confirmation of the drift report is completion criterion 5.

## Pre-emit frontmatter verification (mandatory)

Before marking the drift report as complete, run a field-by-field diff of the YAML frontmatter between every source skill and its converted target. Record as a table in the drift report:

| Skill | Field | Source value | Target value | Status |
|---|---|---|---|---|
| `<name>` | `name` | ... | ... | ✓ preserved / ✗ changed |
| `<name>` | `description` | ... | ... | ✓ preserved / ✗ changed |
| `<name>` | `invocable` | ... | ... | ✓ preserved / ✗ changed (REQUIRES register entry) |
| `<name>` | `compatibility` | ... | ... | ✓ preserved / ✗ changed |
| `<name>` | (any other field) | ... | ... | ... |

**Rule:** any frontmatter field whose target value differs from the source without a corresponding approved-additions entry is a **drift-report error**. Resolve either by registering the change or by reverting the target to match source. The drift report CANNOT be signed off while a mismatch is outstanding.

This step catches:
- Silent field drops (source has `invocable: false`, target omits the field → default flips to `true`, a behavior change)
- Silent field additions (target adds `invocable: false` not in source)
- Typos in preserved fields (source `compatibility: "A"`, target `compatibility: "A."` — extra period)
- Description rephrasing that was not intentional

The verification table is a standalone section in the drift report, placed before §Summary.

## Location

```
docs/plans/YYYY-MM-DD-<target-skill-name>-conversion/drift-report.md
```

(Same directory as the plan doc for that conversion project.) Use `templates/drift-report-template.md` as the empty scaffold.

## Structure

```markdown
# Drift Report: <target-skill-name>

**Source:** <source-skill-path>, version <N>
**Target:** <target-path>
**Conversion dimensions:** <list>

## Frontmatter verification

| Skill | Field | Source value | Target value | Status |
|---|---|---|---|---|
| <name> | name | ... | ... | ✓ / ✗ |
| <name> | description | ... | ... | ✓ / ✗ |
| <name> | invocable | ... | ... | ✓ / ✗ |
| <name> | compatibility | ... | ... | ✓ / ✗ |

(Every ✗ must link to an approved-additions entry. Unregistered ✗ = drift-report error, sign-off blocked.)

## Phase 5 status

| Target skill | Status | Evidence |
|---|---|---|
| <skill> | PASS / SKIPPED / USER-PENDING | `phase5-result.md` / source frontmatter quote / `phase5-handoff.md` |

## Summary

- Total approved additions: N
  - Phase 1 (initial): X (minor: Xm, moderate: Xmo, heavy: Xh)
  - Phase 4 (empirical): Y (minor: Ym, moderate: Ymo, heavy: Yh)
- Total rejected proposals: Z (Phase 4 only)
- Overall drift verdict: LOW / MODERATE / HIGH
- Rationale for verdict: <one paragraph>

## Phase 1 approved additions

<Full table from approved-additions register, filtered to Phase 1 source-phase>

## Phase 4 approved additions

<Full table from register, filtered to Phase 4>

(If Phase 4 was SKIPPED: state so, rest of this section omitted)

## Phase 4 rejected proposals

<Table: date, location, proposal-summary, rejection-reason>

(If Phase 4 was SKIPPED: omit)

## Drift verdict detail

Describe the qualitative character of the drift:
- Are additions concentrated in one area, or distributed?
- Is the drift mostly clarifying (low risk) or structural (high risk)?
- Does the converted skill still carry the source's authorial voice, or has it shifted?

This is the human-judgment section. One short paragraph.
```

## Drift-verdict heuristic

- **LOW:** All additions minor; total count < 5; no heavy additions
- **MODERATE:** Mix of minor/moderate; 1-3 heavy additions, or total count 5-15
- **HIGH:** ≥ 4 heavy additions, or total count ≥ 15, or heavy additions concentrated in the same section

The verdict is advisory. If LOW, the user likely accepts silently. If MODERATE or HIGH, the user should review whether to accept, pare back, or re-run with stricter Phase 4 settings.

## User confirmation

At completion, present the drift report. Ask:

> "Drift report is at <path>. Please review and confirm you accept the aggregate drift before I finalize. Modification, rollback, or re-run are all options."

Only mark the conversion complete after explicit confirmation.

## Anti-patterns

- Omit rejected proposals from the report (loses the "considered and declined" audit trail)
- Let the impact-summary section stand empty (drift cannot be surveyed at a glance)
- Mark the conversion complete without user confirmation of the drift report
- Skip the pre-emit frontmatter verification table (silent field drops or typos go undetected)
- Record Phase 5 as PASS when the actual state is USER-PENDING (user has not yet run the fresh-session test)
