# Approved-Additions Register

## Purpose

A single audit-grade ledger recording every piece of content that is in the target but not in the source. Without this, the source-compare check in Phase 2 would flag legitimate additions as defects and cause infinite loops.

## Schema

| Column | Required | Description |
|---|---|---|
| Added-on | yes | Absolute date (YYYY-MM-DD) |
| Location | yes | Section / heading / line anchor in the target |
| Summary | yes | One-line gist of the addition |
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

## Anti-patterns

- Adding content to target without approving the register entry first
- Treating a registered entry as source-compare material (infinite loop)
- Approving with empty Reason or Impact (audit becomes useless)
- Merging Phase 1 and Phase 4 entries (loses provenance, loses the signal of "how much empirical-tuning drifted the skill")
