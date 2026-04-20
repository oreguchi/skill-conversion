# <Target Skill Name> Conversion Plan

> **For agentic workers:** Use `superpowers:executing-plans` or `superpowers:subagent-driven-development`. This plan was scaffolded by `skill-conversion` — the 8 Phase 0 declarations below MUST all be filled in before proceeding to Phase 1.

**Goal:** Convert <source-skill-name> into <target-skill-name> across the declared dimensions.

**Architecture:** Apply the skill-conversion flow (Phase 0 → Phase 5) with fixpoint review and approved-additions register.

---

## Phase 0: Scope Declaration

### 1. Source skill

- Path: `<path-to-source/SKILL.md>`
- Version: <N>
- References included: <list of references/*.md>

### 2. Dimension declaration

| Dimension | Source | Target | Changes? |
|---|---|---|---|
| Technical | | | yes / no |
| Locale | | | yes / no |
| Agent | | | yes / no |

At least one row must be "yes".

### 3. Conversion-rule tables

#### Technical (if applicable)

<paste table from references/conversion-dimensions.md structure, filled in>

#### "No equivalent" constructs (technical)

<paste table>

#### Locale (if applicable)

<terminology table + style rules>

#### Agent (if applicable)

<frontmatter schema map + tool-name map + invocation-convention notes>

### 4. Scope-strip list

| Topic | Reason for strip |
|---|---|

### 5. Approved-additions register (initially empty)

| Added-on | Location | Summary | Reason | Source-phase | Impact | Approver |
|---|---|---|---|---|---|---|

### 6. Evaluation-scenario skeleton

#### Scenario 1 (central case)

- Situation: <one paragraph>
- Requirement checklist:
  - [ ] [critical] <item>
  - [ ] <item>
  - [ ] <item>

#### Scenario 2 (edge)

<same structure>

#### Scenario 3 (optional)

<same structure>

### 7. Phase 4 opt-in decision

- [ ] opt-in (run Phase 4 per references/verification-stages.md)
- [ ] opt-out (skip Phase 4). Reason: <reason>

### 8. Cross-agent verification strategy (required if agent dimension is "yes")

- Phase 3 strategy: 3a / 3b / (combination)
- Phase 4 strategy: 4a / 4b / 4c / 4d / (combination)
- Phase 5 strategy: 5a / 5b / 5c / (combination)
- Rationale: <one paragraph>

---

## Phase 1: Initial Translation

<task list per source file>

## Phase 2: Fixpoint Review Loop

<iteration log>

## Phase 3: Code Executability Check

<environment method, results>

## Phase 4: Empirical Prompt Tuning (if opt-in)

<scenarios, iterations, approvals, rejections>

## Phase 5: Auto-Fire Test

<trigger prompts, results>

## Completion

- [ ] Phase 2 zero-change pass
- [ ] Phase 3 PASS (or SKIPPED)
- [ ] Phase 4 converged (or SKIPPED per Phase 0 §7)
- [ ] Phase 5 PASS
- [ ] Drift report generated at `drift-report.md`
- [ ] User confirmation received
