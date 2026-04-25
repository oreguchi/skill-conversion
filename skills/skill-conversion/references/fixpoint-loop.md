# Fixpoint Review Loop (Phase 2)

## Why a fixpoint

A single review pass is not enough. Every fix creates risk of introducing new defects — the loop terminates only when a full review produces **zero changes**. This is a fixpoint in the mathematical sense: the content stops moving under the review operator.

## The three review axes

Run in parallel each iteration via `superpowers:dispatching-parallel-agents`.

### Technical check

**Goal:** Target is technically correct.

**What to verify:**
- Code samples compile / parse / run (deferred to Phase 3 for full execution; this axis catches syntax-level obvious breakage)
- API / library usage matches the target ecosystem's current idioms
- Factual claims (e.g., "VB.NET supports X") are correct

**How it differs from Phase 3:** Phase 3 runs code. This axis reviews code *as content*, catching shallower issues without the cost of build-and-run.

### Prose check

**Goal:** Readable, internally consistent prose.

**What to verify:**
- Terminology consistent (per the locale-dimension terminology table if applicable)
- No contradictions between sections
- Structure flows (section ordering matches reader expectations)
- Anti-patterns of skill writing absent (no preamble, no empty-handed promises, no references before they're introduced)

### Source-compare check

**Goal:** Target is faithful to source — nothing meaningful added, nothing meaningful dropped.

**What to verify:**
- Every section/idea in source has a corresponding section/idea in target (unless in scope-strip list)
- Target has no content absent from source UNLESS that content is registered in the approved-additions register
- Scope-strip list items are absent from target

**Register interaction:** When comparing, load the approved-additions register. **Registered entries are excluded from the comparison** — they are expected not to match the source. This is the mechanism that lets approved additions exist without triggering infinite loops.

## Loop rule

```
iteration i:
  run tech-check, prose-check, source-compare in parallel
  collect findings
  if any axis produced one or more change findings:
    apply all findings
    (if source-compare detected unapproved content in target:
        halt, ask user; on approval update register)
    goto iteration i+1
  else:
    record "zero-change pass #1"
    if convergence requires 2 consecutive zero-change passes:
      goto iteration i+1
    else:
      converge
```

## Convergence criterion

- **Default:** one zero-change pass is enough
- **High-stakes skills (recommended uplift):** require two consecutive zero-change passes. Use this when the skill will be widely used or when source-compare findings have been frequent in earlier iterations

### v1.0 convergence stance

In v1.0 the convergence criterion is **always high-stakes** (two consecutive zero-change passes), independent of the active Conversion Profile. The reasoning: profile is a tool for shaping what the skill covers, not for relaxing the quality gate that proves it converges.

Profile-driven differences appear instead in Phase 4 opt-in control, the approved-additions threshold, and the Catalog reference range. See `references/profile-system.md`.

## When to halt for user input

- Source-compare finds content in target that is absent from source AND not in the register → stop, present to user, apply their decision, resume loop
- Technical check finds a factual claim that requires domain judgment → stop, ask user
- Prose check wants to restructure a section → stop if restructure changes meaning; else apply directly

## Loop-back from other phases

Phase 3 failures return here. Phase 4 approved changes return here. Phase 5 failures (description revision) return here. Each return resets the zero-change pass counter.

## Anti-patterns

- Running the axes sequentially (slow, and axis outputs may contaminate each other)
- Ending the loop after one pass without confirming it was zero-change
- Bundling unrelated fixes into one "fix" — keep each fix small and trackable in the plan-doc history
- Treating a registered entry as a source-compare violation
