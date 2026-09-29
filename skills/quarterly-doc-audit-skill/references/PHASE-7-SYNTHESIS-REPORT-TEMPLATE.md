# Phase 7 — Synthesis Report Template

**Used by:** `quarterly-doc-audit-skill` Step 5 (Phase 7 Synthesis)
**When:** After Phases 1-6 complete (all CC outputs absorbed + Phase 6 decisions resolved)
**Purpose:** Codify the §1-§7 structure of the synthesis report so each quarter produces apples-to-apples deliverables. Phase 8 Systemic Review consumes this report as its primary input — **section numbering is load-bearing**, Phase 8 references sections by number. Do not renumber across quarters.

---

## Format rules

- **Persist to Drive.** Write to `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}.md` via `Filesystem:write_file`. Start as skeleton + `status: IN PROGRESS` when Phases 1-3 close; append via `edit_file` as later phases land. Status changes to `COMPLETE` only after Phase 8 + cascade land (Phase 8 depends on Phase 7 as input, so Phase 7 stays OPEN until Phase 8 report exists).
- **Target ≤500 lines final.** Synthesis is a summary; detail lives in Phase 1-5 reports. If >500 lines, something belongs in a phase report instead.
- **Cross-ref, don't reproduce.** Cite `Phase 2 F#` / `Phase 3 F#` rather than reproducing finding prose verbatim. Readers needing detail follow the cross-ref.
- **Populated vs. stub markers.** Use `_{populated during Phase N}_` in sections that fill later. Never leave an empty `{...}` placeholder in a finalized doc.
- **Every Tier 1 row cites a source phase + finding number.** No "misc" Tier 1 rows.
- **§3 Patterns must distinguish prevention-applied vs. proposed vs. pending** — Phase 8 uses this distinction to determine what still needs systemic work.

---

## Template

Copy the block below into `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}.md`. Fill placeholders as phases land.

````markdown
---
title: HAIOS Doc Audit Q{N} {YEAR} — Synthesis Report
version: {x.y}
updated: {YYYY-MM-DD} (S{session})
status: IN PROGRESS | COMPLETE
purpose: Phase 7 synthesis of the Q{N} {YEAR} doc audit. Aggregates Phases 1-5 CC outputs + Phase 6 Jordan decisions into the primary audit deliverable. Serves as primary input to Phase 8 Systemic Review.
parent: HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-SCOPE.md
depends_on:
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-1-INVENTORY.md
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-2-CONFLICTS.md
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-3-STALENESS.md
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-4-CONSOLIDATION.md
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-5-COMPLIANCE.md
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-6-DECISION-MEMO.md
---

# HAIOS Doc Audit Q{N} {YEAR} — Synthesis Report

> **Status:** {IN PROGRESS | COMPLETE}. {1-2 sentence progress summary — which phases populated, which pending.}

## 1. Executive Summary

**Audit window:** {month} {YEAR}, Sessions S{first}–S{last}
**Scope:** {scope_in one-line summary} ({N} files inventoried)
**Execution:** 8-phase methodology per `quarterly-doc-audit-skill` v{version}

### Headline counts

| Phase | Findings | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|---|
| 2 Conflicts | {N} | {N} | {N} | {N} |
| 3 Staleness | {N} | {N} | {N} | {N} |
| 4 Consolidation | {N} | {N} | {N} | {N} |
| 5 Compliance | {N} | {N} | {N} | {N} |
| **Total** | **{N}** | **{N}** | **{N}** | **{N}** |

### Top-3 most-impactful findings

_Select from §2 Action Table. Rank by combined severity × scope. Each entry: 1-2 sentences referencing source phase F#._

1. **{Pattern name}** — {Cross-phase trace, e.g., "Phase 2 F2/F3/F4 + Phase 3 F#9 all trace to..."}. Prevention/remediation: {applied S{N} | proposed Decision {N} | pending {trigger}}.
2. **{...}**
3. **{...}**

### Preventions applied in-session

Structural fixes that landed during the audit rather than deferring to §7 Cascade.

| Fix | Target | Origin |
|---|---|---|
| {fix name} | {doc} | {phase feedback → Tier 1 approved S{N}} |

---

## 2. Action Table

All actionable findings across phases, sorted by Tier ascending, then by Severity.

| # | Phase | Finding | Severity | Proposed Fix | Owner | Tier | Blocker? |
|---|---|---|---|---|---|---|---|
| {N} | {2/3/4/5} | {F#} | {rating} | {fix} | {COO/CC/Jordan} | {1/2/3} | {y/n} |

---

## 3. Pattern Observations

Recurring themes. Each pattern is a candidate Routing Precedent for `HAIOS/Knowledge/DOC-GOVERNANCE.md`. **Phase 8 Systemic Review consumes this section as input — status labels (✅ PREVENTION APPLIED / ⚠ PROPOSED / ⏳ PENDING) drive Phase 8 formalization lane.**

### Pattern {A/B/C/...} — {name}

**Evidence:** {cross-phase F# references}
**Prevention status:** {✅ applied S{N} | ⚠ proposed via Decision {N} | ⏳ pending {specific trigger}}
**Proposed Routing Precedent for DOC-GOVERNANCE:**
> {1-3 sentence precedent text, imperative mood}

---

## 4. SSOT-REGISTRY-MASTER Deltas

### Version bumps applied in-session

| From | To | Session | Change |
|---|---|---|---|
| v{x} | v{y} | S{N} | {summary} |

### Additions proposed (Tier 1, pending Phase 6)

- `{registry slot}` — {one-line rationale}

### Path updates proposed (Tier 3, Phase 5 inline)

_{populated from Phase 5 Pass 1 findings — path renames / archive-moves affecting registry rows}_

### Removed / superseded

_{populated from Phase 4 + Phase 5 if deprecations approved Phase 6}_

---

## 5. Decision Log — Tier 1 (Jordan resolutions)

Phase 6 memo outputs roll up here. Each row: source | decision | COO recommendation | Jordan resolution.

| # | Source | Decision | COO Recommendation | Jordan Resolution |
|---|---|---|---|---|
| {N} | {Phase F#} | {decision title} | {APPROVE \| MODIFY \| REJECT} — {one-line} | {Jordan's} |

**Recommendation format:** `APPROVE | MODIFY | REJECT` — one-line reason. Full decision memos at `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-6-DECISION-MEMO.md`.

---

## 6. Execution Plan

### In-session — Phase 5 Tier 3 inline

- {list of Phase 5 auto-fixes with doc | change one-liners}

### Queued as separate CC dispatch (Phase N.5 or next session)

- {list with trigger gate}

### Pending Jordan Tier 1 (Decision Log §5)

- Count: **{N}** (resolution SLA per decision-memo entry)

### Deferred to Q{N+1}-{YEAR} audit cycle

- _{any finding Phase 6 explicitly deferred}_

---

## 7. Cascade Updates Post-Report

Scoped to THIS audit's findings. Systemic-level improvements (skill updates, new rules, structural hooks) land via Phase 8 Systemic Review instead.

After this report is finalized + before Phase 8 executes, COO completes:

- [ ] `HAIOS/Knowledge/DOC-GOVERNANCE.md` — append Patterns {A, B, C, ...} to Routing Precedents table. Bump DOC-GOVERNANCE version.
- [ ] `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md` — apply §4 deltas beyond those already landed. Bump version if deltas present.
- [ ] `HAIOS/session-context.md` — FLAGS: remove resolved triggers; add next anchor `Q{N+1}-{YEAR} audit: {month} {YEAR}`.

**After cascade, Phase 8 executes** (per `quarterly-doc-audit-skill` Step 6).

---

## Changelog

| Version | Session | Changes |
|---|---|---|
| {x.y} | S{N} | {summary} |

````

---

## Template vs. actual report: what changes per quarter

| Stable across quarters | Per-quarter |
|---|---|
| Frontmatter schema | Version, updated, status |
| §1-§7 section numbering | Data populated in each |
| Pattern-labeling convention (A/B/C...) | Which patterns are found |
| Cascade checklist categories | Specific docs touched |
| Authority hierarchy language | Number of Tier 1/2/3 items |

Do not renumber sections across quarters. Phase 8 references Phase 7 by §N.

---

## Changelog of this template

| Version | Session | Changes |
|---|---|---|
| 1.0 | 592 | Initial creation. Derived from `DOC-AUDIT-Q2-2026.md` v0.2 structure. Codifies §1-§7 layout, populated-vs-stub markers, prevention-status labels in §3, explicit forward reference to Phase 8. §8 Methodology Retrospective from prior drafts migrates to Phase 8 template. |
