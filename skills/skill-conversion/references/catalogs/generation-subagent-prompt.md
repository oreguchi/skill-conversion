# Catalog Generation Subagent Prompt

> Instructions handed to a subagent during catalog generation in skill-conversion v1.0+. Implements W1' Mechanism 1 (clear classification criteria).

## Your task

Generate a **Catalog** for the target locale + framework combination. The catalog follows the structure of `references/catalogs/catalog-template.md` — exactly two sections: Expected capabilities and Gotchas.

## Tier classification criteria (strict)

### Tier 1 (mandatory)

Decision rule: **the source skill addresses a corresponding capability**, OR "the target-environment equivalent of a capability the source covers".

Example: if the source C# WinForms skill covers "data binding", the VB.NET WinForms catalog Tier 1 includes "BindingSource and INotifyPropertyChanged".

### Tier 2 (recommended for balanced or higher)

Decision rule: the source does not cover it, BUT **at least 2 of the following 4 conditions** hold:

1. The target environment's **official documentation** describes it as a recommended baseline pattern.
2. **Major books** (O'Reilly, Microsoft Press, Apress, etc.) describe it.
3. **Official tutorials** (Microsoft Learn, MDN, etc.) describe it.
4. The README of a major OSS project in the target ecosystem positions it as a standard pattern.

Example for VB.NET WinForms: "UI-thread marshalling", "IProgress(Of T)", "DI form resolution" all qualify as Tier 2.

### Tier 3 (recommended for high-utility)

Decision rule: target-environment-specific **differentiating or advanced** capability. It does not meet the Tier 2 conditions, but specialised books or advanced documentation describe it.

Example for VB.NET WinForms: "PerMonitorV2 high-DPI handling", "an async-timer alternative to Application.DoEvents".

## Gotchas — recording policy

Enumerate target-environment-specific compiler/runtime restrictions across 4 categories: language, type, syntax, API.

Each entry should include:
- A description of the restriction
- The error code (if applicable)
- An alternative approach (a different pattern that achieves the same intent)

## Forbidden

- **Do not classify Tier 2 / Tier 3 by speculation.** If you cannot find official documentation or major-book coverage, exclude it from the catalog (do **not** dump it into Tier 3 as a fallback).
- **Do not split entries to inflate counts.** Prefer one entry that names a coherent capability over five fragments of the same capability.
- **Do not place source-specific capabilities at Tier 1+.** If the source uses an unusual extension, classify it at Tier 2 / Tier 3, not Tier 1.

## Output

Present a draft to the user as `references/catalogs/<lang>-<framework>.md`. The user reviews the Tier classification and the item content, then revises and approves (Mechanism 4 — content-approval gate).
