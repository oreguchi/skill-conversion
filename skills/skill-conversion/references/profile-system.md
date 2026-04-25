# Conversion Profile System (v1.0 specification)

> The "expected value" the user declares in Phase 0 §7, and the ways that declaration mechanically controls each Phase.

## Three-stage Profile

| Profile | Meaning |
|---|---|
| **high-fidelity** (高忠実度) | Source-fidelity first. Preserving the source author's intent outweighs adding target-environment value. |
| **balanced** (バランス) ⭐ default | Fidelity and utility together. Cover the de-facto essentials of the target environment. |
| **high-utility** (高有用性) | Target reader's utility first. Adopt target-environment differentiating capabilities. |

## Four axes of automatic linkage

The profile declaration mechanically determines all four axes below.

| Axis | high-fidelity | balanced | high-utility |
|---|---|---|---|
| **Phase 4** | default on, opt-out allowed (reason required) | forced on | forced on |
| **approved-additions threshold** | strict (A only) | standard (A + B) | permissive (A + B + C) |
| **Catalog reference range** | Gotchas + Tier 1 | Gotchas + Tier 1 + Tier 2 | Gotchas + Tier 1 + Tier 2 + Tier 3 |
| **Phase 2 fixpoint convergence** | always high-stakes (profile-independent) | same | same |

For details:
- approved-additions threshold: `references/approved-additions.md` § 9-cell approval table
- Catalog reference range: `references/catalog-system.md` § Profile-linked reference range
- Phase 2 fixpoint: `references/fixpoint-loop.md`

## Phase 4 opt-out (high-fidelity only)

### Opting out under high-fidelity

1. Select "high-fidelity" in Phase 0 §7.
2. Record a **mandatory reason** in §7.1.
3. State the skip explicitly at the top of the drift report: `Phase 4: SKIPPED by user choice (high-fidelity profile, reason: <reason>)`.

### Opt-out attempted under balanced or high-utility

1. **Halt.**
2. Ask the user to re-select:
   - Switch profile to high-fidelity (which makes opt-out available), OR
   - Run Phase 4 (accept the forced-on).
3. The main agent enforces this at the end of Phase 0 §7.

## Default behaviour and missing values

- **Omitted** → balanced is adopted automatically; warning is logged.
- **Unknown value** → halt and require re-declaration (defends against typos).

## Where the profile is stored

The profile is recorded in the Phase 0 plan-doc §7. Subsequent phases read the plan-doc to obtain the current profile.

## Profile change mid-conversion

To change the profile during an active conversion:

1. Re-declare it in Phase 0 §7 (edit the plan-doc).
2. Restart the conversion from the top — including rebuilding the Phase 0 §6 scenario skeleton.
3. Partial re-evaluation is **not** permitted (it would break profile consistency across phases).
