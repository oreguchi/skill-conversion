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

### On failure: distinguish skill content vs. verification scaffolding

Before fixing, categorize the failure:

- **(a) Skill content** — the broken code block is inside `SKILL.md` or a `references/` file (the skill's own deliverable)
- **(b) Verification scaffolding** — the broken file is under a test-harness / sample-build directory whose sole purpose is to prove the skill's code compiles (e.g., `sample-build/SampleProject/`)

Apply the right response:

- **(a) Skill-content failure:** fix the code in the skill; **return to Phase 2** (skill content changed, fixpoint must restart)
- **(b) Scaffolding failure:** fix the scaffolding; **do not return to Phase 2** (skill content unchanged). Record the scaffolding fix in the Phase 3 record: what was wrong in the harness, in one line, with the file path

**Rule:** never modify skill content to make a scaffolding bug disappear. If a scaffolding fix seems to require a skill-side change, re-read the skill's code block — the bug might genuinely be there (case (a)).

Both categories need an audit trail. The Phase 3 record in the plan doc should list each fix with its category tag `[content]` or `[scaffolding]`.

## Phase 4: Empirical prompt tuning (opt-in)

**Opt-in rationale.** This layer is powerful but carries **drift risk**: a blind subagent will stumble wherever the source author intentionally assumed reader knowledge, and "fixing" that drifts the skill away from source intent. The user decides in Phase 0 whether that trade-off is worth it.

### Opt-out

If the user opts out in Phase 0 §7, skip Phase 4 entirely. Record `Phase 4: SKIPPED by user choice (reason: <reason>)` in the plan doc. Drop Phase 4 from the completion criteria.

### Opt-in: preparation

Finalize the evaluation-scenario skeleton from Phase 0 §6:
- 2-3 scenarios, each a realistic task the skill would be applied to
- Each scenario has a requirement checklist (3-7 items), with at least one `[critical]`-tagged item
- At least one scenario is a mid-difficulty "central case", at least one is an edge case

Apply the Phase 0 §6 bias-control rule: scenarios are derived from the source skill's `description` + "When to Use This Skill" section + `compatibility` field, NOT from its body content (headings, code examples, reference-file sections). Body-derived scenarios only test what you already know the skill covers, reducing Phase 4 to a self-consistency check.

### Opt-in: checklist commitment rule (anti-goalpost-moving)

The `[critical]`-tagged items in each scenario's checklist are **committed at Phase 0 §6 / Phase 4 preparation** and **MUST NOT be modified during Phase 4 execution**. Specifically:

- A `[critical]` item cannot be downgraded to non-critical after a scenario run
- A `[critical]` item cannot be removed after a scenario run
- A `[critical]` item's wording cannot be softened after a scenario run

**If a scenario run fails a `[critical]` item, the default outcome is FAIL.** The scorer then proposes a skill-content fix (which registers as a Phase 4 approved-addition and loops back to Phase 2). The scorer does NOT revise the checklist after seeing the run result.

**Legitimate checklist revision path.** If after running you realize the checklist itself was flawed (e.g., a `[critical]` item was phrased ambiguously, or applied to a scenario-path the skill's design explicitly allows either way), the only protocol-valid response is:

1. Halt Phase 4 for that scenario
2. Return to Phase 0 §6 / preparation
3. Document the scenario-design flaw in the plan doc
4. Revise the scenario and checklist **before** any result is known
5. Re-dispatch the subagent with the revised checklist and score fresh

This is deliberately expensive — that's the point. Retroactive checklist tweaking after seeing the output is how blind tests quietly become rubber stamps.

### Opt-in: what "blind" means structurally

Each subagent dispatch in Phase 4 must satisfy all three conditions. If any condition is unmet, flag the run as "semi-blind" in the empirical results file and treat the findings as a self-consistency check rather than an effectiveness check.

1. **Fresh instance.** The subagent has no prior context from the main conversion session. A subagent reused from a prior turn is NOT blind.

2. **Content isolation.** The target skill's file paths (SKILL.md, `references/*.md`) may be named in the prompt so the subagent knows where to Read. The skill's **body content** (headings, examples, prose) MUST NOT be pasted into the prompt — the subagent must discover content only by Read-ing the files itself. Otherwise the main agent's summary of the skill is what's being tested, not the skill.

3. **Scorer separation (recommended, not mandatory).** If a separate scorer subagent is affordable, dispatch one to compare the executor subagent's output against the `[critical]` checklist. If only self-scoring by the main agent is feasible, record this explicitly in the empirical results file:

   > Scorer: main agent (self-scored). Bias direction: lenient toward the skill's Phase-1 translation choices.

   Self-scoring is not disqualifying, but it must be **named** so the drift-report reader can weight the Phase 4 evidence correctly.

### Opt-in: execution

Invoke `empirical-prompt-tuning` with the target skill and scenarios. Each iteration dispatches fresh subagents — every dispatch must satisfy the three blind-conditions above. Collect both self-reported findings and measured metrics (success, precision, steps, duration, retries) per the `empirical-prompt-tuning` convention.

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

### Skills with `invocable: false`

If the target skill's frontmatter contains `invocable: false`, the skill does not participate in natural-language auto-fire — it is loaded only by explicit `Skill(<name>)` invocation. Phase 5 is therefore **SKIPPED by design** for such skills.

Record in the plan doc:

```
Phase 5: SKIPPED (invocable: false preserved from source)
```

No trigger-prompt drafting, no fresh-session test, no failure loop. The SKIP is the correct outcome.

**Exception — invocable drift.** If source and target `invocable` values differ (source `true` or omitted ↔ target `false`, or vice versa), the field change is itself a content change that requires an approved-additions entry with justification. Phase 5 outcome follows the target value; the drift must be registered, not silently absorbed.

### Environment (for invocable:true skills)

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

### User-handoff requirement (mandatory for invocable:true targets)

Phase 5 is **unrunnable inside the conversion session** — same-session execution produces context-contamination false positives. The conversion's skill-side work cannot PASS Phase 5; only the user can, in a fresh session. To make this handoff structural rather than discretionary, the skill MUST perform three actions before declaring completion:

1. **Write per-skill trigger prompts.** For every target skill with `invocable: true` (or the field omitted), emit `phase5-<skill>.md` in the plan directory. Each file contains:
   - 2-3 natural-language prompts a real user would type
   - Expected evidence: `Skill(<name>)` tool call appearing in transcript before the assistant's answer
   - FAIL-response procedure (return to Phase 2 with description fix)

   For `invocable: false` skills, emit a stub `phase5-<skill>.md` stating the SKIP reason with the source-frontmatter quote.

2. **Write a cross-skill handoff guide.** Use `templates/phase5-handoff-template.md` to produce `phase5-handoff.md` in the plan directory. This is the single document the user opens to perform Phase 5. It must contain: the SKIP/action table, the installation/fresh-session/CWD procedure, per-skill PASS criteria, and the FAIL-to-Phase-2 loopback procedure.

3. **Announce the handoff in chat.** Before any completion-criterion declaration, output a user-facing message containing:
   - The literal string `Phase 5: USER-PENDING`
   - The path to `phase5-handoff.md`
   - A one-sentence reminder that Phase 5 cannot be run in the current session

   This announcement MUST NOT be suppressed even if all other phases converged with zero drift. The completion-criterion check must treat Phase 5 as `USER-PENDING` until the user returns a result.

**Anti-pattern:** burying the handoff only in the drift report's "Outstanding Work" paragraph. The user may never reach that section; the handoff must surface in chat output AND as its own dedicated file.

### User-reported outcomes

After the user runs the handoff procedure, they report back one of:

- `PASS` with `phase5-result.md` filed (all trigger prompts fired the correct skill) — Phase 5 passes
- `FAIL` with specific prompts that did not fire — skill re-enters Phase 2 with description revision registered as a Phase 4 approved-addition

### Agent-dimension target

Phase 5 cannot be run inside Claude Code for non-Claude-Code targets. Follow the strategy chosen in Phase 0 §8 — typically:
- Generate a `fire-test/<skill-name>.md` test-procedure doc
- User installs on the target agent, runs the procedure, returns PASS/FAIL + observations
- The conversion completes on the user's reported result
