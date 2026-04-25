# Approved-Additions Register

## Purpose

A single audit-grade ledger recording every piece of content that is in the target but not in the source. Without this, the source-compare check in Phase 2 would flag legitimate additions as defects and cause infinite loops.

## Category (v1.0 NEW)

One of the two axes of approval judgement. Classifies the **nature** of an addition.

| Category | Nature | Criterion |
|---|---|---|
| **A. Fidelity-preserving** | Correction needed to preserve source fidelity | Addresses compiler/runtime constraints that prevent retention of source content; ensures syntactic equivalence |
| **B. Environment-essential** | De-facto essential in the target environment | Maps to Catalog Tier 1 + Tier 2. Generally recognised as "without this the skill would be incomplete" |
| **C. Differentiating** | Target-environment-specific differentiating capability | Maps to Catalog Tier 3. Less ubiquitous than Tier 2, or oriented to specific use cases |

Category is judged by a subagent or the main agent and recorded in the register. See `references/catalog-system.md` for the Catalog ↔ Category mapping.

## Schema

| Column | Required | Description |
|---|---|---|
| Added-on | yes | Absolute date (YYYY-MM-DD) |
| Location | yes | Section / heading / line anchor in the target |
| Summary | yes | One-line gist of the addition |
| **Category** | **yes (v1.0 NEW)** | **A / B / C — see § Category** |
| Reason | yes | Why this addition exists (environment-specific, user-requested, subagent-stumbled-on-missing-info, etc.) |
| Source-phase | yes | `Phase 1` or `Phase 4` — which phase introduced it |
| Impact | yes | `minor` / `moderate` / `heavy` — degree of drift from source intent |
| Approver | yes | User identifier who approved it |

An empty register at the start of Phase 0 is fine. Entries only appear after the user explicitly approves an addition. Use `templates/register-template.md` as the empty scaffold.

## Lifecycle

1. **Phase 0 initialization:** empty table written to plan doc (or a sibling file `approved-additions.md`)
2. **Phase 1 trigger:** during initial translation, if conversion rules don't fully cover source content and the translator believes new content is needed, halt and propose it to the user
3. **Phase 4 trigger:** during empirical-prompt-tuning, if a blind subagent stumbles on missing information and the proposed fix is new content, halt and propose it
4. **User approval:** individual, not batch. The form for Phase 4 is stricter (see `verification-stages.md` §Phase 4)
5. **Register entry:** on approval, add a row; on rejection, do not add content to target
6. **Phase 2 interaction:** source-compare loads the register and excludes registered locations from comparison
7. **Completion:** register is frozen, copied to the drift report (see `drift-report.md`)

## Why impact-assessment matters

The column `Impact` forces the user to confront how far the addition takes the skill from its source intent. Accepting many `heavy` additions signals the conversion has drifted; the drift report summarizes this at the end.

## Example entries

```
| 2026-04-20 | SKILL.md §3.2 | Option Strict On as default | Required for VB.NET code samples to compile reliably | Phase 1 | moderate | oreguchi |
| 2026-04-21 | SKILL.md §4 top | "Prerequisites" block | Blind subagent needed to know prerequisites up front | Phase 4 | minor | oreguchi |
```

## 9-cell approval table (v1.0 NEW)

Automatic judgement matrix by Category × Profile. Combined with Impact (minor / moderate / heavy) yields 27 cells.

| Category × Impact | high-fidelity | balanced | high-utility |
|---|---|---|---|
| A. Fidelity · minor | auto-approve | auto-approve | auto-approve |
| A. Fidelity · moderate | auto-approve | auto-approve | auto-approve |
| A. Fidelity · heavy | user approval required | user approval required | user approval required |
| B. Essential · minor | user approval required | auto-approve | auto-approve |
| B. Essential · moderate | **rejected (default)** | user approval required | auto-approve |
| B. Essential · heavy | **rejected** | user approval required | user approval required |
| C. Differentiating · minor | **rejected** | user approval required | auto-approve |
| C. Differentiating · moderate | **rejected** | **rejected** | user approval required |
| C. Differentiating · heavy | **rejected** | **rejected** | user approval required |

### Cell behaviours

- **auto-approve** — subagent or main-agent judgement is recorded as approved in the register; no user prompt, but post-hoc review is permitted.
- **user approval required** — present the approval form (subject / detail / rationale / impact / category / verdict) to the user; on approval, append to the register.
- **rejected (default)** — the addition is **not adopted** because it conflicts with the profile's stated expectation. It is NOT added to the register; instead it is logged in the drift-report's "Profile-rejected proposals" section.

### Handling rejected cases

If a profile cell rejects an addition, the addition is **not adopted** even when a Phase 4 blind subagent reports the addition as "useful for the target reader". This is intentional: the profile declaration represents the user's expectation and is honoured.

If the user wants a different outcome, they must **re-declare the profile in Phase 0 §7** (which restarts the conversion from the top — partial re-evaluation is not supported because it would make the profile inconsistent across phases).

## Anti-patterns

- Adding content to target without approving the register entry first
- Treating a registered entry as source-compare material (infinite loop)
- Approving with empty Reason or Impact (audit becomes useless)
- Merging Phase 1 and Phase 4 entries (loses provenance, loses the signal of "how much empirical-tuning drifted the skill")
