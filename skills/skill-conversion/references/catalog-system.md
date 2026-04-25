# Catalog System (v1.0 specification)

> The single specification for catalog generation, approval, persistence, and Scenario linkage. This file is the authoritative source for SKILL.md §X.1 / §X.2.

## Structure

Each catalog file (`references/catalogs/<lang>-<framework>.md`) has **two fixed sections**:

- Expected capabilities — Tier 1 / Tier 2 / Tier 3 (3 levels)
- Gotchas — required reading for every profile

See `references/catalogs/catalog-template.md`.

## Tier classification criteria

See `references/catalogs/generation-subagent-prompt.md` (subagent instructions used during catalog generation).

## Generation and approval flow (W1' = Mechanisms 1 + 4 + 5)

### Phase 0 §X.1 — Catalog status check

1. The main agent reads `references/catalogs/INDEX.md`.
2. The main agent decides whether the matching combination (lang + framework) is persisted.
3. The main agent presents the result to the user, who chooses reuse or regenerate.
4. Reuse → adopt the existing catalog into Phase 0 §6 (skip generation).
5. Regenerate → proceed to §X.2.

### Phase 0 §X.2 — Catalog generation and two-stage approval

#### Stage 1 — content approval (Mechanism 4)

1. Dispatch a subagent (model: sonnet recommended). Pass `references/catalogs/generation-subagent-prompt.md` as the prompt.
2. The subagent generates a catalog draft following `catalog-template.md`.
3. The user reviews Tier classification and entries.
4. After edits or as-is, the user approves → catalog is used for Phase 0 §6 (in memory).

#### Stage 2 — persistence approval (NEW)

1. The main agent asks the user whether to persist.
2. Persist = yes → save to `references/catalogs/<lang>-<framework>.md` and append a row to `INDEX.md`.
3. Persist = no → keep in memory only; discard after Phase 5 ends.
4. Record the persistence status in the drift report's "Catalogs used" section.

## Profile-linked reference range

| Profile | Catalog reference range |
|---|---|
| high-fidelity | Gotchas + Tier 1 |
| balanced | Gotchas + Tier 1 + Tier 2 |
| high-utility | Gotchas + Tier 1 + Tier 2 + Tier 3 |

The "Gotchas" section is read regardless of profile.

## Catalog ↔ Scenario linkage

### Phase 0 §6 — when drafting the scenario skeleton

1. The main agent reads the catalog's "Expected capabilities" up to the profile's Tier range.
2. Each scenario's Requirement checklist embeds items from that range.
3. Each embedded item is tagged `[Tier N]`, making the catalog reference point explicit.

### Phase 4 — blind eval scorer

1. A blind subagent runs each scenario and produces an answer.
2. The scorer (main agent or an independent subagent) mechanically checks:
   - Coverage of `[critical]`-tagged checklist items (v0.2 behaviour)
   - Coverage of `[Tier N]`-tagged checklist items (v1.0 NEW)
3. If a Tier-tagged item is uncovered, propose an additional scenario or escalate to approved-additions as a candidate.

## Phase 1–2 gotcha auto-merge

### Linkage with the §3b "no-equivalent" table

The "Gotchas" section of a catalog adopted in Phase 0 is **auto-merged** into the §3b no-equivalent table during Phase 1–2. It is not maintained separately by hand.

Concretely, when the main agent builds the conversion-rule tables, the catalog's gotchas are pulled in as §3b entries. On duplicate, the catalog version wins.

## INDEX.md auto-maintenance

On persistence approval, the main agent automatically:

1. Appends a row to `references/catalogs/INDEX.md`.
2. Fills "Catalog / Target combination / Last updated / Coverage summary".
3. Coverage summary reuses the subagent's generation summary, or the main agent writes a one-liner.

Deletion is manual (user records it in the drift report).
