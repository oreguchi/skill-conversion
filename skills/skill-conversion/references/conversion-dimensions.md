# Conversion Dimensions

## Overview

A conversion is described by which of the three dimensions change. At least one must change; more than one is a "compound" conversion.

## Technical dimension

**What it covers:** Programming language, framework, platform, runtime version.

**Examples:**
- C#/.NET → VB.NET
- React → Vue
- Python → Go
- .NET 6 → .NET 8 (same language, different framework version)

**Required output — conversion-rule table:**

| Column | Content |
|---|---|
| Source construct | Source-language syntax or idiom |
| Target construct | Target-language equivalent |
| Notes | Use only when behavior differs or is unavailable |

**Also required — "no equivalent" table:**

| Source construct | Why no target equivalent | What to do |
|---|---|---|
| e.g., C# `record` | VB.NET lacks primary-constructor record sugar | Use `Structure` + `ReadOnly` properties + hand-written equality, document in prose |

## Locale dimension

**What it covers:** Natural language, script, cultural context.

**Examples:**
- EN → JA
- EN → ZH (Simplified)
- JA → EN

**Required output — terminology consistency table:**

| Source term | Target term | Notes |
|---|---|---|

**Required output — style rules:**

Document the target-language norms the converted skill must follow:
- Full-width vs half-width (Japanese)
- Punctuation (。、 vs . , vs 。,)
- Katakana conventions (long-vowel mark, loanword choice)
- Capitalization (English title case vs sentence case)

## Agent dimension

**What it covers:** The runtime agent that will load and execute the converted skill.

**Supported targets:**
- Claude Code (default, home base)
- Copilot CLI
- Gemini CLI
- Codex

**Required output — frontmatter schema map:**

| Claude Code field | Target-agent equivalent | Notes |
|---|---|---|
| `name` | (varies) | May require different format or key name |
| `description` | (varies) | Trigger semantics may differ |

**Required output — tool-name map:**

| Claude Code tool | Target-agent tool | Notes |
|---|---|---|
| `Read` | (e.g., `read_file` in target) | |
| `Bash` | (e.g., `run_shell`) | |
| `Skill` | (no equivalent or different invocation) | |

Pre-built references live at the bottom of this file; check target-agent docs for any fields/tools that have changed since.

**Required output — invocation-convention notes:**

How the target agent activates skills (auto-discovery, slash command, explicit activate call, etc.). Document this because the Claude Code auto-fire semantics may not carry over cleanly.

**Agent-specific-feature handling:**

Features that exist on one agent but not another (Claude Code: Task, Agent, TodoWrite, Hooks, Slash Commands, MCP) need explicit scope-strip or explanatory notes.

## Compound conversions

When multiple dimensions change together, each dimension's tables are produced independently; then a **compound-review step** checks for interactions (e.g., a Japanese technical term naming a construct that doesn't exist in the target language).

## Schema templates

Fully worked examples of each table live in `references/templates/` (not this file).
