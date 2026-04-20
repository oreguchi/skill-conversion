# skill-conversion — Plain-English Overview

## In one sentence

A plugin that **converts existing Claude Code skills to a different language, locale, or target agent without silent drift** — every deviation from source is individually approved and logged.

## Why it exists

The Claude Code ecosystem has many excellent skills, but most ship in English, target Claude Code specifically, and use one programming language in their examples. People want to:

- Translate them to Japanese / Chinese / other locales
- Port them to Copilot CLI, Gemini CLI, or Codex
- Re-target code examples to their stack (C# → VB.NET, Python → Go, etc.)

If you simply prompt "translate this skill to Japanese", you'll hit these failure modes:

- **Silent additions** — the model inserts "helpful" extra content that wasn't in the source
- **Silent drops** — hard-to-translate parts quietly disappear
- **Terminology drift** — the same concept gets multiple translations within one doc
- **No diff trail** — you can't tell what changed or why

This plugin solves those structurally, not by asking the model to "be careful".

## Three conversion dimensions

| Dimension | What changes | Example |
|---|---|---|
| **Technical** | Programming language or framework | C# → VB.NET, Python → Go, React → Vue |
| **Locale** | Natural language | English → Japanese, English → Chinese |
| **Agent** | Target runtime agent | Claude Code → Copilot CLI / Gemini CLI / Codex |

All three are independent. You can combine freely — e.g., "English Claude Code TypeScript skill → Japanese Copilot CLI Go skill" is a valid 3-dimension conversion.

## Four core mechanisms

### 1. Conversion rule tables

Before any translation, Phase 0 produces a table per active dimension. Example for Python → Go:

| Python | Go |
|---|---|
| `def foo():` | `func foo()` |
| `class Bar:` | `type Bar struct{}` |
| `try/except` | `if err != nil` |

Locking these down up front prevents ad-hoc per-occurrence decisions and keeps the conversion consistent.

### 2. Approved-additions register

Anything that ends up in the target but wasn't in the source requires explicit user approval, logged with reason and impact:

| Added-on | Location | Summary | Reason | Impact | Approver |
|---|---|---|---|---|---|
| 2026-04-20 | §3.2 | `Option Strict On` as default | Required for VB.NET code examples to compile | moderate | oreguchi |

The audit trail is a first-class artifact, not a side effect.

### 3. Fixpoint review loop

Immediately after translation, three review axes run in parallel:

1. **Technical check** — syntax, APIs, factual claims
2. **Prose check** — terminology consistency, readability, structure
3. **Source-compare check** — target ↔ source for missing or unexplained content

Any single change in any axis restarts all three. The loop continues **until one full pass produces zero changes** (two consecutive zero-change passes for high-stakes skills).

### 4. Drift report

At completion, the register and the axis results feed into a drift report:

- Phase 1 approved additions
- Phase 4 approved additions (if opt-in)
- Phase 4 rejected proposals (the "considered and declined" audit trail)
- Impact summary (minor / moderate / heavy counts)
- Overall verdict: **LOW** / **MODERATE** / **HIGH**

The conversion is not complete until the user confirms the drift report.

## Six-phase workflow

```
Phase 0  Scope Declaration     ← declare all 8 required items up front
  ↓
Phase 1  Initial Translation   ← apply rule tables mechanically
  ↓
Phase 2  Fixpoint Loop         ← 3 axes in parallel, until zero-change
  ↓
Phase 3  Code Executability    ← does the converted code actually build/run?
  ↓
Phase 4  Empirical Tuning      ← opt-in. Blind-subagent effectiveness evaluation
  ↓
Phase 5  Auto-fire Test        ← fresh session verifies the description triggers
  ↓
Complete Drift Report → user confirmation
```

## Self-validated (meta-application)

This plugin's v0.1.0 shipped after running itself through its own Phase 4. Evidence:

- 3 scenarios × 3 iterations = 9 blind-evaluator runs
  - S1: Python → Go (technical only)
  - S2: English → Japanese (locale only)
  - S3: English Claude Code → Japanese Copilot CLI + Go (all three dimensions)
- Precision 100% across every iteration
- Zero retries across every iteration
- Convergence reached at Iter3 (zero triggering signals)
- Final drift verdict: **LOW**

All findings from the evaluation were handled via the skill's own individual-approval form — F1–F4 applied to SKILL.md, U3–U9 as clarifications. Every change is on the record.

## How to use

Talk to it in natural language:

- "Convert this C# skill to VB.NET"
- "Translate this skill to Japanese"
- "Port this Claude Code skill to Copilot CLI in Go"
- Combine freely: "Translate to Japanese and port to Gemini CLI"

The plugin will elicit Phase 0's 8 declarations, generate a plan doc, then walk Phase 1–5 with user gates at every content addition.

## Prerequisites

| Plugin | Role |
|---|---|
| `superpowers` | Planning, parallel-agent, and verification sub-skills |
| `empirical-prompt-tuning` | Blind-evaluator engine for Phase 4 |

## Installation

```
/plugin install superpowers
/plugin install empirical-prompt-tuning
/plugin marketplace add https://github.com/oreguchi/skill-conversion.git
/plugin install skill-conversion@skill-conversion-marketplace
```

## Who should use this

- Plugin authors wanting to produce localized / ported variants of their skills
- Teams who need an **auditable** conversion process ("why is this sentence here?")
- People who accept **quality over speed** — the flow has multiple review iterations

## When not to use

- **Authoring a new skill from scratch** → use `superpowers:writing-skills` instead
- **Minor edits** (typo fix, one-sentence change) → direct edit is faster
- **Non-skill Markdown** (README, docs, specs) → this plugin is tuned for skill format specifically

## License

MIT
