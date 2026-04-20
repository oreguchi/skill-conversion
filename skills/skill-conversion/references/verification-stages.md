# Verification Stages (Phase 3 / 4 / 5)

The three verification layers, each catching a different class of failure.

| Stage | Catches | Cost | Cardinality |
|---|---|---|---|
| Phase 3: Code executability | Shallow code errors (syntax, type, API misuse) | Cheap, deterministic | Mandatory (skippable only if skill has no code) |
| Phase 4: Empirical prompt tuning | Ambiguity, missing context, structural problems as seen by a blind agent | Expensive, probabilistic | **Opt-in** (user decides in Phase 0) |
| Phase 5: Auto-fire test | Discoverability — does the description actually trigger the skill? | Medium | Mandatory |

Run in order: fail fast on cheap checks before spending on expensive ones.

## Phase 3: Code executability

**Rule:** Every code block in the target must be confirmed executable in the target environment. The method varies; the check itself does not.

### Method-by-target mapping

| Target | Method |
|---|---|
| Compiled languages (VB.NET, C#, Go, Rust, Kotlin…) | Build a small sample project including the code; run `build` / `compile` |
| Dynamic languages (Python, JavaScript, Ruby…) | Syntax check (`python -m py_compile`, `node --check`) + execute where feasible |
| Shell scripts | `shellcheck` or equivalent static analyzer + sandbox run |
| Markup (HTML, SQL, YAML) | The appropriate schema validator / parser |
| Skill contains no code blocks | SKIP. Record "Phase 3: SKIPPED (no code blocks)" in plan doc |

### Agent-dimension adjustment

If the target is for a non-Claude-Code agent, any agent-specific tool-invocation snippets (e.g., a Gemini-CLI activate call) must also be syntactically validated per that agent's documented schema. See `cross-agent-verification-menu.md` for the strategy matrix.

### On failure

Fix the code. Return to Phase 2 — fixpoint must restart because content changed.

## Phase 4: Empirical prompt tuning (opt-in)

**Opt-in rationale.** This layer is powerful but carries **drift risk**: a blind subagent will stumble wherever the source author intentionally assumed reader knowledge, and "fixing" that drifts the skill away from source intent. The user decides in Phase 0 whether that trade-off is worth it.

### Opt-out

If the user opts out in Phase 0 §7, skip Phase 4 entirely. Record `Phase 4: SKIPPED by user choice (reason: <reason>)` in the plan doc. Drop Phase 4 from the completion criteria.

### Opt-in: preparation

Finalize the evaluation-scenario skeleton from Phase 0 §6:
- 2-3 scenarios, each a realistic task the skill would be applied to
- Each scenario has a requirement checklist (3-7 items), with at least one `[critical]`-tagged item
- At least one scenario is a mid-difficulty "central case", at least one is an edge case

### Opt-in: execution

Invoke `empirical-prompt-tuning` with the target skill and scenarios. Each iteration dispatches fresh subagents. Collect both self-reported findings and measured metrics (success, precision, steps, duration, retries) per the `empirical-prompt-tuning` convention.

### Opt-in: individual change-approval form

For every change proposed, present to the user:

```
【Phase 4 Change Proposal】
Target location: SKILL.md §X.Y or references/<name>.md §Y.Z
Summary:        <one line>
Detail:         <full text of proposed change>
Rationale:      <which subagent-reported unclear-point led to this>
Impact:         minor / moderate / heavy
                (minor = clarification of existing sentence;
                 moderate = adds missing context;
                 heavy = adds a whole new section or shifts structure)
Verdict:        approve / reject
```

### Opt-in: change classification

- **Clarification of existing sentence** (impact: minor) — apply as normal Phase 2 change; DO NOT register
- **Added information / new section** (impact: moderate / heavy) — register in approved-additions (source_phase = `Phase 4`); then loop to Phase 2

### Opt-in: consolidated diff on convergence

When the empirical-tuning loop converges, present a single consolidated diff of all Phase 4 changes so the user can ratify the aggregate effect. Allow rollback at this gate.

### Convergence criterion (opt-in only)

Two consecutive iterations with:
- Zero new self-reported unclear-points
- Precision improvement ≤3 points
- Step-count change within ±10%
- Duration change within ±15%

### On change

Any change returns to Phase 2 (content changed, fixpoint restarts).

## Phase 5: Auto-fire test

**Rule:** The skill's description must actually trigger the skill on realistic user prompts.

### Environment

- Fresh Claude Code session (not the conversion session — context contamination invalidates the test)
- Different working directory from the one where the skill was authored (prevents the session from having the skill's text in context by accident)
- Skill installed to `~/.claude/skills/` OR to its target plugin already loaded

### Procedure

For each evaluation scenario, write a **natural-language trigger prompt** — what a real user would type to solicit help on that scenario. Submit the prompts. Observe whether the Skill tool auto-invokes the target skill.

Record per scenario: prompt, fired/not-fired, which skill fired, whether the response addressed the scenario substantively.

### Pass criterion

All scenarios fire the target skill correctly, and the skill's response addresses the scenario.

### On failure

Revise the `description` field. Return to Phase 2.

### Agent-dimension target

Phase 5 cannot be run inside Claude Code for non-Claude-Code targets. Follow the strategy chosen in Phase 0 §8 — typically:
- Generate a `fire-test/<skill-name>.md` test-procedure doc
- User installs on the target agent, runs the procedure, returns PASS/FAIL + observations
- The conversion completes on the user's reported result
