# Phase 5 User-Handoff Guide

**Conversion project:** <project-name>
**Issued:** <YYYY-MM-DD>
**Why this document exists:** Phase 5 (auto-fire test) cannot be run inside the conversion session — same-session execution produces context-contamination false positives. The test must be performed by the user in a fresh session. This guide is the single entry point for that handoff.

## Target skills covered by this handoff

| target skill | `invocable` | Phase 5 | User action required? |
|---|---|---|---|
| <skill-1> | `false` | SKIPPED (by design) | No |
| <skill-2> | (omitted → default true) | Requires fresh-session test | **Yes** |
| <skill-3> | true | Requires fresh-session test | **Yes** |

**Rule:** skills with `invocable: false` preserved from source are SKIPPED by design — they do not participate in natural-language auto-fire. No test is required or meaningful for those.

## User procedure (for skills requiring action)

### 1. Install the skill

Copy or link the converted skill to `~/.claude/skills/<skill-name>/`, or install the target plugin if the skill ships inside a plugin bundle.

Expected result: running `/plugin` or listing `~/.claude/skills/` shows the skill.

### 2. Open a fresh Claude Code session

- **Must be a new session**, not a continuation of the conversion session
- **Must start in a different working directory** from where the skill was authored (this prevents the session from ingesting the skill's source text as ambient context)

Example:

```
cd ~/
claude
```

### 3. Run the trigger prompts

For each skill requiring action, open the corresponding per-skill guide:

- `phase5-<skill-1>.md` — 2-3 natural-language trigger prompts for `<skill-1>`
- `phase5-<skill-2>.md` — 2-3 natural-language trigger prompts for `<skill-2>`

Copy each prompt verbatim into the fresh session. **Do not paraphrase.**

### 4. Observe for PASS evidence

Before the assistant responds with an answer, the transcript must show a `Skill(<skill-name>)` tool call. This is the PASS evidence — it confirms the `description` field triggered auto-fire correctly.

**PASS criterion per skill:** all trigger prompts (2-3) cause the skill to auto-fire. Miss even one → FAIL.

### 5. Record the outcome

Create `phase5-result.md` in the conversion project's plan directory with:

```markdown
# Phase 5 Result

**Tester:** <your name>
**Date:** <YYYY-MM-DD>
**Fresh-session CWD:** <path>

| skill | prompt 1 | prompt 2 | prompt 3 | verdict |
|---|---|---|---|---|
| <skill> | fired/not-fired | fired/not-fired | fired/not-fired | PASS/FAIL |

## Notes

<any observations — which prompt triggered it, any unexpected Skill tool was invoked instead, etc.>
```

## FAIL handling

If any skill FAILs (a trigger prompt did not cause auto-fire), the `description` field is not aligned with how real users phrase requests. Return to Claude in the conversion session with:

> Phase 5 FAIL for `<skill>`: prompt `"<prompt>"` did not trigger the skill. Transcript excerpt: `<paste>`.

Claude's response should be:

1. Register the description revision as a Phase 4 approved-addition (auto-fire discoverability is an effectiveness property)
2. Revise the `description` field
3. Return to Phase 2 (fixpoint restart, since skill content changed)
4. Re-emit this handoff guide and repeat the test

## Completion criterion gate

The conversion cannot be declared complete while this handoff is outstanding. In the drift report, Phase 5 must carry one of:

- `PASS` (user-reported, with `phase5-result.md` linked)
- `SKIPPED (invocable: false preserved from source)` for all target skills
- `USER-PENDING` if the user has not yet reported back

`USER-PENDING` is a valid state during the handoff window but blocks `verification-before-completion` from returning a clean completion.
