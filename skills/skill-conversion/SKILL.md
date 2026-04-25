---
name: skill-conversion
description: "Convert an existing Claude Code skill (or other agent's skill) into a new variant by transforming one or more dimensions: programming language/framework (e.g., C# → VB.NET, React → Vue), natural language locale (e.g., EN → JA), or target agent (e.g., Claude Code → Copilot CLI / Gemini CLI / Codex). Any combination is supported. Use when asked to port, translate, adapt, or re-target an existing skill. Requires superpowers and empirical-prompt-tuning plugins."
---

# skill-conversion

## Prerequisites

This skill has **hard dependencies** on two other plugins. If either is missing, stop and ask the user to install them first.

| Plugin | Role |
|---|---|
| `superpowers` | Provides `writing-plans`, `subagent-driven-development`, `dispatching-parallel-agents`, `verification-before-completion` — all called during the conversion flow |
| `empirical-prompt-tuning` | Called in Phase 4 (opt-in) to evaluate the converted skill's effectiveness via blind subagent execution |

**Installation order (user-facing):**

```
/plugin install superpowers
/plugin install empirical-prompt-tuning
/plugin marketplace add <path-to>/plugin/skill-conversion
/plugin install skill-conversion@skill-conversion-marketplace
```

**Runtime check:** Before entering Phase 0, verify both plugins are available. If not, halt and return the installation instructions above.

## When to use

Use this skill when the user asks to:

- Convert an existing skill to a different programming language or framework (e.g., "convert this C# skill to VB.NET", "port this React skill to Vue")
- Translate an existing skill to a different natural language (e.g., "translate this English skill to Japanese")
- Port a skill to run on a different agent runtime (e.g., "adapt this Claude Code skill for Copilot CLI")
- Any combination of the above

The scope is **always an existing skill as input**. This is not for creating skills from scratch.

## When NOT to use

- **Creating a new skill from scratch** → use `superpowers:writing-skills` instead
- **Minor edits to an existing skill** (typo fix, one sentence rewrite) → edit directly; a full conversion run is overkill
- **Non-skill Markdown documents** (README, docs, specs) → this skill is tuned for the skill format specifically

## Dimensions

Every conversion is described by its dimensions. See `references/conversion-dimensions.md` for full definitions.

| Dimension | What changes | Example |
|---|---|---|
| **Technical** | Programming language, framework, or platform | C#/.NET → VB.NET, React → Vue, Python → Go |
| **Locale** | Natural language, script, or cultural context | EN → JA, EN → ZH |
| **Agent** | Target runtime agent | Claude Code → Copilot CLI / Gemini CLI / Codex |

At least one dimension must change. Dimensions can combine freely (e.g., Claude Code EN C# → Gemini CLI JA VB.NET is a 3-dimension conversion).

## Phase overview

```
Phase 0: Scope Declaration          Profile declaration, Catalog status check / generation
                                     / two-stage approval, conversion-rule tables,
                                     scope-strip list, approved-additions register init,
                                     evaluation-scenario skeleton (Tier-linked)
    ↓
Phase 1: Initial Translation         Apply conversion rules to source files
                                     (parallelizable via subagent-driven-development)
    ↓
Phase 2: Fixpoint Review Loop        3 axes (tech / prose / source-compare) in parallel;
                                     loop until zero changes in one full pass
    ↓
Phase 3: Code Executability Check    Fast deterministic gate for code blocks
                                     (method varies by target environment)
    ↓
Phase 4: Empirical Prompt Tuning     Opt-in. If enabled, blind-subagent effectiveness
                                     check; any adopted change loops back to Phase 2
    ↓
Phase 5: Auto-Fire Test              Fresh session / isolated directory; confirm the
                                     description triggers the skill as intended
    ↓
Completion → Drift Report            Summarize every deviation from source
```

See `references/` for phase details.

## Phase 0: Scope Declaration

Before writing any output, declare the 8 required items in a fresh plan doc:

1. **Source skill** — full path(s), version, and list of included reference files. Source skills use one of several layout patterns; enumerate according to the pattern:
   - **Standard layout** (`SKILL.md` + `references/*.md` subdirectory): list every `*.md` inside `references/`.
   - **Flat layout** (`SKILL.md` and companion `*.md` files share a directory): list every companion `*.md` in the same directory.
   - **`@`-link style** (`SKILL.md` body references `@file.md`): list every `@`-linked file.
   - **Single-file** (no companions): write "(none)".
   
   In every case, the listed files are **conversion targets in Phase 1** (they ship with the skill). Links to *other* plugins' skills are out of scope and must not be listed here.
   
   Also declare the **target skill name**: kebab-case, `<source-name>-<marker>` form is idiomatic (e.g., `writing-plans-ja`, `systematic-debugging-go`, `tdd-go-ja`).
2. **Dimension declaration** — for each of technical/locale/agent: source value, target value, or "not changing"
3. **Conversion-rule tables** — one per active dimension. **Read `references/conversion-dimensions.md` first** — it defines the column schema for each dimension's table. Do not invent a schema.
4. **Scope-strip list** — what to remove entirely (not convert), with reason per entry. If nothing is stripped, write "(none)" plus a one-line rationale.
5. **Approved-additions register** — initialized empty, see `references/approved-additions.md` and use `references/templates/register-template.md` as the scaffold
6. **Evaluation-scenario skeleton** — 2-3 scenarios for Phase 4 (finalized in Phase 4 if opt-in). Each scenario must list a target and a requirement checklist; tag the load-bearing items `[critical]`. Full prose is acceptable at Phase 0 — a bare one-liner is not.

    **Bias-control rule.** Scenarios MUST be derived from:
    - (a) The source skill's **declared use cases** — `description` text, "When to Use This Skill" section, `compatibility` field
    - (b) **Realistic user prompts** — what a user unfamiliar with the skill's internals would actually type

    Scenarios MUST NOT be derived by studying the source skill's body content (headings, code examples, reference-file sections). Doing so bakes in confirmation bias: the scenarios only test what you already know the skill covers, making Phase 4 a self-consistency check rather than an effectiveness check. If you must read the body to understand the source, **draft scenarios first from description + when-to-use alone**, then re-check after reading whether your scenarios accidentally leak body knowledge and revise if so.

    **Catalog reference (v1.0 NEW):** when drafting the scenario skeleton, embed the Tier-N expected-capability items from the active Catalog (Tier range determined by §7 profile) into each Requirement checklist. Tag each embedded item with `[Tier N]` to make catalog reference points explicit. See `references/catalog-system.md`.
7. **Conversion Profile declaration** (v1.0) — declare the conversion's expected-value level in 3 stages:
   - `高忠実度` (high-fidelity)
   - `バランス` (balanced)  ← default when omitted (warning logged)
   - `高有用性` (high-utility)

   Profile mechanically determines 4 axes: **Phase 4 opt-in state**, **approved-additions threshold**, **Catalog reference range**, and (Phase 2 fixpoint convergence is always high-stakes regardless of profile). Unknown values halt for re-declaration.

   **Phase 4 opt-out:** allowed only when profile = high-fidelity, and the reason MUST be recorded in §7.1. Opt-out attempts under balanced / high-utility halt and force re-selection. See `references/profile-system.md` for full semantics.
8. **Verification strategy** — if any agent dimension is non-identity, walk the user through `references/cross-agent-verification-menu.md` and record the choice per Phase 3 / 4 / 5. If the Agent dimension is identity (source-agent = target-agent), mark §8 `N/A — Agent dimension = identity` and the default Phase 3/4/5 strategies apply; the menu walk-through is skipped.

**Reading policy during Phase 0:** when an item above points to a `references/*.md` file (items 3, 5, 7, 8), open and read it before filling that item. The SKILL.md body is an index; the referenced files contain the schemas and decision matrices.

Call `superpowers:writing-plans` to create the plan doc at `docs/plans/YYYY-MM-DD-<target-skill-name>-conversion.md`. Use `references/templates/plan-doc-template.md` as the starting scaffold. **If the conversion run includes iterative empirical testing** (e.g., multiple Phase 4 iterations), use the subdirectory form `docs/plans/YYYY-MM-DD-<target-skill-name>-conversion/` and place per-iteration files inside it (e.g., `iter1-phase0-plan.md`, `empirical-results.md`, `drift-report.md`).

**Do not proceed to Phase 1 until all 8 items are written and the user has confirmed.**

### §X.1: Catalog status check (v1.0)

At Phase 0 entry, scan persisted catalogs and present them to the user:

```
[Phase 0 §X.1: Catalog status]
Persisted catalogs:
- vb-net-winforms.md (last updated 2026-04-25)
- python-go.md (last updated 2026-04-10)

Matches for this conversion: vb-net-winforms.md
Reuse? [Y/n]
```

- Reuse → adopt the catalog into Phase 0 §6 / Phase 1
- Regenerate → proceed to §X.2

### §X.2: Catalog generation and two-stage approval (v1.0)

**Stage 1 — content approval (gate):**

A subagent generates a catalog draft following the Tier classification criteria in `references/catalogs/generation-subagent-prompt.md`. The user reviews Tier classification and item content, then approves (with optional edits). Once approved, the catalog is used for Phase 0 §6 (in memory).

**Stage 2 — persistence approval (NEW):**

After content approval, the user is asked whether to persist:

```
[Catalog persistence confirmation]
Persist this catalog?

□ Persist  → save to references/catalogs/<lang>-<framework>.md + auto-update INDEX.md
□ One-shot → memory only, discarded after Phase 5
```

See `references/catalog-system.md` for the full lifecycle.

## Phase 1: Initial Translation

For each source file (SKILL.md + each `references/*.md`):

1. Apply conversion-rule tables from Phase 0
2. Apply scope-strip list (remove these topics entirely)
3. **Never invent equivalents.** If a source construct has no target equivalent, state "no direct equivalent" in prose; do not fake a lookalike
4. **Never silently add content not in source.** If new content seems necessary, halt and propose it to the user:
   - Describe the proposed addition
   - State why
   - On user approval, register it in the approved-additions register (see `references/approved-additions.md`)
   - On rejection, omit it

Multiple files can run in parallel via `superpowers:subagent-driven-development`.

## Phase 2: Fixpoint Review Loop

**Core quality gate.** Loop until a full review pass produces zero changes.

Each iteration runs 3 review axes in parallel (see `references/fixpoint-loop.md` for full details):

1. **Technical check** — syntax, API correctness, factual claims
2. **Prose check** — terminology consistency, readability, structure
3. **Source-compare check** — target ↔ source for missing or extra content. **Entries in the approved-additions register are excluded from this check.**

Any single-axis change finding → restart all three axes. Zero changes across one full pass → converge (for **high-stakes skills, require two consecutive zero-change passes**). A skill is **high-stakes** if any of: (a) it encodes a methodology intended to be executed verbatim (e.g., TDD, debugging flow), (b) the converted skill will be published as a plugin for third-party use, or (c) the user explicitly flags it as high-stakes in Phase 0.

If source-compare finds content in target not in source and not in the register, halt and ask the user:
- Approve + add to register → loop continues
- Reject → remove content → loop continues

Use `superpowers:dispatching-parallel-agents` to run the three axes concurrently per iteration.

## Phase 3: Code Executability Check

Every code block in the target must be verifiable in the target environment. The **method is environment-specific, but the check is mandatory**.

See `references/verification-stages.md` §Phase 3 for environment→method mapping. If the target skill contains no code blocks at all, skip this phase and note it in the plan doc.

On failure: identify whether the bug is in **(a) skill content** (a code block inside SKILL.md or a `references/` file) or **(b) verification scaffolding** (a file under a test-harness / sample-build directory that exists only to prove the skill's code compiles):

- **For (a):** fix the code in the skill, **return to Phase 2** (skill content changed, fixpoint must restart)
- **For (b):** fix the scaffolding, **do not return to Phase 2** (skill content unchanged). Log the fix in the Phase 3 record with a one-line explanation of what was wrong in the harness

Never modify skill content to make scaffolding bugs disappear. If a scaffolding fix seems to require changing the skill's code block, re-read the block: the bug might genuinely be there.

## Phase 4: Empirical Prompt Tuning (opt-in)

Per Phase 0 §7, Phase 4 is **opt-in**. If opt-out, skip this phase entirely and note `Phase 4: SKIPPED by user choice (reason: ...)` in the plan doc.

If opt-in:

1. Finalize the evaluation-scenario skeleton from Phase 0 (2-3 scenarios, each with `[critical]`-tagged requirement checklist)
2. Invoke `empirical-prompt-tuning` against the target skill
3. For every change proposed, present the individual-approval form (subject / detail / rationale / impact / verdict). See `references/verification-stages.md` §Phase 4 for the form
4. On approval:
   - "Clarification only" (impact: minor) → treat as Phase 2 change, no register entry
   - "Adds information" or "adds a new section" (impact: moderate/heavy) → register in approved-additions (source `Phase 4`), then loop to Phase 2
5. At convergence, present a **single consolidated diff** of all Phase 4 changes for a final ratification pass. Allow rollback

See `references/verification-stages.md` §Phase 4 for convergence thresholds.

## Phase 5: Auto-Fire Test

Install the completed skill to `~/.claude/skills/` or to the target plugin location.

**Run the test in a fresh session, different directory** — same-session tests leak context and produce false positives.

For each evaluation scenario, draft a natural-language trigger prompt (what a real user would type). Run in the fresh session; verify the skill auto-fires (Skill tool invocation appears). On failure, revise the description field and return to Phase 2.

For agent-dimension conversions (target ≠ Claude Code), Phase 5 cannot be executed inside Claude Code. Follow the strategy selected in Phase 0 §8 — typically the skill produces a fire-test document at `fire-test/<skill-name>.md` for the user to execute on the target agent and report back.

### Skills with `invocable: false`

If the target skill's frontmatter contains `invocable: false`, the skill does not participate in natural-language auto-fire — it is loaded only by explicit `Skill(<name>)` invocation from another skill or from the user. Phase 5 is therefore **SKIPPED by design** for such skills.

Record in the plan doc as:

```
Phase 5: SKIPPED (invocable: false preserved from source)
```

No trigger-prompt drafting, no fresh-session test, and no failure-retry loop is required or appropriate. The SKIP is the correct outcome, not a gap in coverage.

**Exception — invocable drift.** If the source has `invocable: true` (or the field omitted, which defaults to true) but the target has `invocable: false`, or vice versa, this is a content change that REQUIRES an approved-additions entry. Phase 5 outcome follows the target value, but the drift must be registered and justified — not silently absorbed.

### User-handoff requirement (mandatory)

Phase 5 cannot be PASSed within the conversion session. The skill MUST perform the following three actions before declaring the conversion run finished — these are completion-gate items, not optional:

1. **Emit per-skill trigger-prompt files.** For every target skill with `invocable: true` (or the field omitted), write `phase5-<skill>.md` in the plan directory with:
   - 2-3 natural-language prompts a real user would type
   - Expected observable evidence (`Skill(<name>)` call appearing in transcript before the assistant answers)
   - FAIL-response procedure (return to Phase 2 with description fix)

   For `invocable: false` skills, write a stub `phase5-<skill>.md` stating "SKIPPED (invocable: false preserved from source)" with the source-frontmatter quote as evidence.

2. **Emit a cross-skill handoff guide.** Write `phase5-handoff.md` in the plan directory using `references/templates/phase5-handoff-template.md`. This single file is the ONE document the user needs to open to complete Phase 5. It must contain the SKIP/action table, the user procedure, per-skill PASS criteria, and the FAIL-to-Phase-2 loopback procedure.

3. **Announce the handoff explicitly in chat.** Before any completion declaration, output a user-facing message containing:
   - The literal string `Phase 5: USER-PENDING`
   - The path to `phase5-handoff.md`
   - A one-sentence reminder that Phase 5 cannot be run in the current session

   This message MUST NOT be suppressed even if all other phases have converged with zero drift. Completion-criterion check (via `superpowers:verification-before-completion`) must treat Phase 5 as `USER-PENDING` — not PASS — until the user explicitly reports back.

**Anti-pattern:** burying the Phase 5 handoff only inside the drift report's "Outstanding Work" paragraph. The user may not read that far; the handoff must surface both in the chat output and as a dedicated file.

## Completion criteria

All of the following must hold:

1. Phase 2 fixpoint loop converged (one full zero-change pass, or two for high-stakes)
2. Phase 3 code executability PASS (or SKIPPED with reason logged)
3. Phase 4 empirical-prompt-tuning converged (opt-in) OR SKIPPED by user choice
4. Phase 5 auto-fire test: each target skill resolves to exactly one of — **(a) user-reported PASS** from a fresh-session test (for invocable:true targets), **(b) SKIPPED (invocable: false preserved from source)**, or **(c) user-reported PASS** for agent-dimension targets. A conversion with multiple targets is allowed to mix (a)(b)(c) freely; what is NOT allowed is leaving any target in `USER-PENDING` — that state blocks completion until the user reports back
5. **Drift report generated and user-confirmed**

Use `superpowers:verification-before-completion` to run the final check.

## Drift report

At completion, emit a drift report at `docs/plans/<same>/drift-report.md` that summarizes every deviation from source:

- Phase 1 approved additions (initial-conversion judgment calls)
- Phase 4 approved additions (effectiveness improvements; only if Phase 4 was opt-in)
- Phase 4 rejected proposals (audit trail of what was considered and declined)
- Impact-assessment summary (minor / moderate / heavy counts)
- Overall drift verdict

See `references/drift-report.md` for schema and generation rules.

## Anti-patterns (do not do these)

**Content:**
- Inject information not in source without user approval
- Fake a lookalike target construct when no equivalent exists — say "no direct equivalent" instead
- Leave scope-strip topics partially converted

**Process:**
- End the Phase 2 loop without a zero-change pass
- Skip code executability check because it's tedious
- Decide Phase 4 opt-in/out unilaterally — ask the user in Phase 0
- Run the auto-fire test in the same session (context contamination)
- Make unilateral decisions for agent-dimension verification strategy — walk the user through the menu
- Bury the Phase 5 user-handoff inside the drift report — surface it in chat AND as a dedicated `phase5-handoff.md`
- Retroactively tweak a `[critical]` checklist item after seeing the subagent's output (goalpost moving)
- Derive Phase 4 scenarios from the target skill's body content instead of its description + when-to-use (confirmation bias)
- Modify skill content to make a sample-build / test-harness bug disappear

**Approved-additions register:**
- Treat a registered entry as source-compare material (would cause infinite loops)
- Approve with empty reason or empty impact-assessment
- Merge Phase 1 and Phase 4 entries (lose provenance)
- **Batch-approve Phase 4 changes** — each must be individually reviewed

**Drift report:**
- Omit rejected Phase 4 proposals (audit loses the "considered and declined" trail)
- Omit the impact-assessment summary (drift cannot be surveyed at a glance)
