# Release Notes

## 1.0.1 — 2026-04-27

**Patch release: install-blocking bug fix.** No spec or feature changes.

### Bug fix

- **`.claude-plugin/plugin.json` was missing the `skills` array** — the manifest did not enumerate `./skills/skill-conversion`, so Claude Code could not resolve the plugin's component path during install. Symptom: `Failed to install: Source path does not exist: <marketplace clone path>`.
- The same gap existed in v0.2.0 but went unnoticed because earlier Claude Code releases auto-discovered the `skills/` directory. Newer Claude Code versions require the explicit array.

### Files changed

- `.claude-plugin/plugin.json` — added `"skills": ["./skills/skill-conversion"]`
- `.claude-plugin/marketplace.json` — version bump 1.0.0 → 1.0.1
- `RELEASE_NOTES.md` — this entry

No backward-compat impact for already-installed v1.0.0 users. Update procedure (Claude Code has no single-step `/plugin update`):

```
/plugin uninstall skill-conversion@skill-conversion-marketplace
/plugin marketplace update skill-conversion-marketplace
/plugin install skill-conversion@skill-conversion-marketplace
```

Or enable auto-update for the marketplace in `/plugin` UI → Marketplaces tab.

---

## 1.0.0 — 2026-04-25

**Major release: stable.** Introduces the Conversion Profile + Catalog System + Phase-control trinity.

### New features

- **Conversion Profile (3 stages)** — `high-fidelity` / `balanced` / `high-utility`. Declared at Phase 0 §7.
- **Catalog System** — captures target-environment-specific "expected capabilities" (Tier 1 / 2 / 3) and "gotchas" in a 2-section file. Generation + two-stage approval (content approval, then persistence approval) + Scenario linkage.
- **9-cell approval table** — approved-additions are auto-judged by Category × Profile (auto-approve / user approval required / rejected).
- **Phase 3 source-confirmation** — adopted APIs are recorded with a 4-tier officiality level (Official / Standard / Peripheral / Unknown). Peripheral / Unknown adoptions force the both-pattern annotation in SKILL.md.
- **Drift-report extensions** — Profile declaration header, Catalogs used, Phase 3 adopted-API roster, Profile-rejected proposals.

### Breaking changes

- Phase 0 §7 changed from "Phase 4 opt-in decision" to "Conversion Profile declaration".
- The approved-additions register now has a required Category column.
- Phase 4 opt-in/out is profile-linked (only `high-fidelity` may opt out, with a mandatory reason in §7.1).
- The Catalog concept is new (generation, approval, persistence flow added).
- Phase 3 reporting now includes an Adopted-API roster.

See `skills/skill-conversion/references/migration-v02-to-v10.md` for migration steps.

### Out of scope (deferred to v1.x)

- Phase 4 baseline_variant feature
- Auto-PR posting for catalog generation (community workflow)
- Multi-language INDEX.md (e.g., bilingual EN/JA)

---

## 0.2.0

(Initial-release feature set: Phase 0–5 base flow, approved-additions register, cross-agent verification menu.)
