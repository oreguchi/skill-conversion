# Migration Guide: skill-conversion v0.2 → v1.0

> Guidance for conversions in progress under v0.2.0, or for v0.2.0 plan-docs that should continue under v1.0.0.

## Recommended approach

**Strongly recommended: finish in-progress conversions under the v0.2.0 spec.** Reasons:

- The v1.0.0 mechanisms (Profile, Catalog) are designed to be adopted from Phase 0 onwards.
- A mid-flight switch is confusing because §7's meaning changes (Phase 4 opt-in decision → Profile declaration).
- The plan-doc shape and the v1.0 shape are not fully compatible.

## Already-completed conversions

Artifacts produced under v0.2.0 (drift-report, approved-additions register, etc.) remain readable under v1.0.0. **No breaking impact.**

Caveats:

- v0.2.0 register has no Category column → it appears as "unclassified" when displayed under v1.0.
- The 9-cell approval table does not retroactively apply to v0.2.0 register entries (past judgements stand).

## Switching to v1.0.0 (when needed)

To migrate an in-progress conversion to v1.0.0:

### 1. Rewrite plan-doc §7 as a Profile declaration

Old form:

```markdown
### 7. Phase 4 opt-in decision
- [x] opt-in
- [ ] opt-out
```

New form:

```markdown
### 7. Conversion Profile declaration
- [ ] high-fidelity
- [x] balanced  ← honour the existing opt-in by choosing balanced
- [ ] high-utility
```

### 2. Add the Catalog status sections

Insert §X.1 / §X.2 immediately after §7. Either run catalog generation + two-stage approval, OR (e.g., under high-fidelity opt-out) declare "no catalog used" — and decide how to proceed.

### 3. Add a Category column to the approved-additions register

Retroactively classify every existing register entry as A / B / C. Use the definitions in `references/approved-additions.md` § Category.

### 4. Apply the 9-cell table prospectively

Existing approved entries are honoured (no change). Apply the 9-cell table from the next addition onwards.

### 5. Reflect the drift-report template diff

Append the new sections introduced by `references/templates/drift-report-template.md` to the existing drift-report:

- Conversion Profile (header)
- Catalogs used
- Phase 3 adopted-API roster
- Profile-rejected proposals

## Support

If a switch causes issues, file an issue against skill-conversion.
