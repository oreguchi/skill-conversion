# Cross-Agent Verification Strategy Menu

Use this menu when the conversion's agent dimension is non-identity (target runtime ≠ Claude Code).

In Phase 0 §8, walk the user through this menu. Select one (or a combination) per Phase 3, 4, 5. Record the selections in the plan doc along with the user's rationale.

## Phase 3: Code executability

| # | Strategy | Description |
|---|---|---|
| 3a | As-is | Code is agent-agnostic in most cases. Verify in the target language / framework environment normally |
| 3b | Plus agent-call syntax check | If the skill includes examples of agent-specific tool calls (e.g., a Gemini-CLI `activate_skill` invocation), also statically validate those calls against the target agent's documented schema |

**Default recommendation:** 3a. Add 3b if the skill has concrete agent-invocation examples.

## Phase 4: Empirical prompt tuning

**Constraint:** Claude Code cannot dispatch subagents on foreign agents. Options for opt-in Phase 4:

| # | Strategy | Description | Drift risk |
|---|---|---|---|
| 4a | Skip (opt-out) | Don't run Phase 4 at all | Low (fidelity wins) |
| 4b | Structural-review mode | Invoke `empirical-prompt-tuning` in structural-review mode on Claude Code — static consistency / clarity check only, no subagent dispatch | Low-to-moderate |
| 4c | Surrogate execution | Run full empirical tuning on Claude Code even though target is different. Valid only for skill content that's agent-agnostic; agent-specific sections will produce noisy findings | Moderate (Claude-Code-shaped subagent's stumbles may not apply to the target) |
| 4d | User manual-use report | User actually uses the skill on the target agent, reports friction / unclear points; those become the iteration signals instead of subagent findings | Low (but high user burden) |

**Default recommendation:** 4b as baseline. For high-importance skills, 4b + 4d (structural review catches obvious issues cheaply; user manual-use catches real-world gaps).

## Phase 5: Auto-fire test

| # | Strategy | Description |
|---|---|---|
| 5a | Test-procedure doc + user execution | Generate `fire-test/<skill-name>.md` with trigger prompts and observation checklist. User runs on target agent in fresh session, records and reports results. **Mandatory for all agent-dimension targets** |
| 5b | Description-form early check | On Claude Code, statically check that the description's trigger vocabulary aligns with typical user phrasings. Not a replacement for 5a but catches obvious description-field defects cheaply |
| 5c | Skip | Description is reviewed but no actual trigger test. **Not recommended** |

**Default recommendation:** 5a (required) + 5b (cheap early gate).

## Menu walkthrough procedure

In Phase 0 §8:

1. Ask the user: "Target agent is [X]. For each of Phase 3, 4, 5, choose a strategy (defaults: 3a, 4b, 5a+5b). State any reason to deviate."
2. Record the selected strategies + rationale in the plan doc:

   ```
   Cross-agent verification strategy:
   - Phase 3: 3a
   - Phase 4: 4b + 4d
   - Phase 5: 5a + 5b
   Rationale: <user's reason>
   ```
3. Subsequent phases read these selections and execute accordingly.

## Evolving agent specs

Each target agent's skill format and tool names change over time. Never reuse a prior conversion's mapping blindly — verify each target-agent's current spec at the start of Phase 0.

## v1.0 supplementary note

When the agent dimension is non-identity AND the profile is balanced or high-utility (Phase 4 forced on), strategy selection from this menu is mandatory. Cross-reference `references/profile-system.md` together with this file.

If the profile is high-fidelity AND the agent dimension is non-identity AND Phase 4 is opt-out, the cross-agent verification strategy applies only to §3 / §5 (the §4 row is N/A).
