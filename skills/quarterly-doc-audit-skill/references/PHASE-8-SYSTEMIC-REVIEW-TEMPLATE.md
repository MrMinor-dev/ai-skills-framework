# Phase 8 — Systemic Review Template

**Used by:** `quarterly-doc-audit-skill` Step 6 (Phase 8 Systemic Review)
**When:** After Phase 7 synthesis report closes (Phase 7 is the primary input)
**Purpose:** Codify the per-component review structure so Phase 8 produces apples-to-apples deliverables each quarter. Phase 8 translates audit findings into concrete improvement proposals for COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade rules, DOC-GOVERNANCE routing precedents, Anti-Patterns, and automation tools. Closes the feedback loop from findings back into the system that generated them.

---

## Format rules

- **Persist to Drive.** Write to `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md` via `Filesystem:write_file`. Status starts `IN PROGRESS` during Phase 8 execution; moves to `COMPLETE` when §3 Tier 1 proposals are staged in `session-context.md` + §4 Tier 2/3 executions finish.
- **Target ≤400 lines final.** This is a structured review, not a synthesis narrative. Keep rows per-component-table tight — the per-component-type tables carry the load.
- **Every review row cites at least one finding.** Row without a finding source = not a review row, delete it. "Nothing to do here" components get a 1-line entry stating so with a scan note; they don't get full review rows.
- **Tier 1 proposals are standalone.** Do NOT append Phase 8 Tier 1 items to the Phase 6 decision memo. They queue as standalone entries in `session-context.md` NEXT SESSION, resolved in the NEXT session after the audit closes. Rationale: Phase 8 runs after Phase 6 already closed; re-opening Phase 6 to accept new decisions breaks the phase sequencing.
- **Tier 2/3 items auto-execute in-session.** Phase 8 COO has Tier 2 authority for: skill version bumps, cascade rule additions, AP promotions (Watch→Active), AP retirements, AP new-entries (where <20 lines), DOC-GOVERNANCE Routing Precedent formalizations. Anything requiring new doc creation >100 lines, new skill creation, new hook deployment, or backward-incompatible change = Tier 1.

---

## Staleness pre-check (mandatory before execution)

Before starting Phase 8, confirm these inventories are current:

- [ ] `AOS/Skills/SKILLS-ROADMAP.md` — all session-level skill edits since last audit reflected
- [ ] `AOS/CC-Config/MANIFEST.md` — CC skills + rules + hooks + agents list current
- [ ] `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md` — reflects in-session bumps from this audit
- [ ] `AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md` — anti-pattern  + Watch-list current

If any inventory is more than a few sessions stale relative to recent session-context.md, REFRESH it first. Stale inventory invalidates Phase 8's coverage claim — "we reviewed all COO skills" isn't true if SKILLS-ROADMAP is missing 3 recent entries.

---

## Template

Copy the block below into `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md`. Fill placeholders as the review proceeds.

````markdown
---
title: Doc Audit Q{N} {YEAR} — Phase 8 Systemic Review
version: {x.y}
updated: {YYYY-MM-DD} (S{session})
status: IN PROGRESS | COMPLETE
purpose: Phase 8 systemic review of the Q{N} {YEAR} doc audit. Translates Phase 1-7 findings into concrete improvement proposals across COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade rules, DOC-GOVERNANCE routing precedents, Anti-Patterns, and automation tools. Tier 1 proposals queue standalone for NEXT session; Tier 2/3 execute in-session.
parent: HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}.md
depends_on:
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}.md (Phase 7 synthesis — primary input)
  - HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-{2-5}-*.md (detail sources)
  - AOS/Skills/SKILLS-ROADMAP.md (COO skills inventory)
  - AOS/CC-Config/MANIFEST.md (CC components inventory)
  - HAIOS/Architecture/SSOT-REGISTRY-MASTER.md
  - HAIOS/Knowledge/DOC-GOVERNANCE.md
  - AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md
---

# Doc Audit Q{N} {YEAR} — Phase 8 Systemic Review

> **Status:** {IN PROGRESS | COMPLETE}. {1-sentence progress summary.}

## 1. Executive Summary

**Input:** `DOC-AUDIT-Q{N}-{YEAR}.md` v{x} (Phase 7 synthesis, {N} findings, {N} Tier 1 resolutions)
**Review pass executed:** {Opus, S{session}, {N} component types}
**Outputs produced:** {N Tier 1 proposals (standalone §3), M Tier 2/3 executions (§4), P cross-quarter patterns (§5)}

### Headline counts

| Component Type | Reviewed | Gaps Found | Tier 1 Proposals | Tier 2/3 Executed |
|---|---|---|---|---|
| COO Skills | {N} | {N} | {N} | {N} |
| CC Skills + Rules + Hooks | {N} | {N} | {N} | {N} |
| SSOT-REGISTRY Cascade Rules | {N existing} | {N new surfaced} | {N} | {N} |
| DOC-GOVERNANCE Routing Precedents | {N existing} | {N new surfaced} | {N} | {N} |
| Anti-Patterns + Watch-list | {N AP + N W} | {N promotions + N new} | {N} | {N} |
| Tools / n8n Workflows | {N} | {N automation ops} | {N} | {N} |
| **Totals** | — | **{N}** | **{N}** | **{N}** |

### Top systemic issues this quarter

_Select from §2 review rows. Rank by combined (finding count × component type leverage). Each entry: 1-2 sentences referencing finding sources._

1. **{issue name}** — {which component fell short, which findings trace to it, proposed resolution scope}.
2. **{...}**
3. **{...}**

---

## 2. Per-Component Review

One subsection per component type. Review row template per subsection is specific to that component.

### 2.1 COO Skills

**Inventory source:** `AOS/Skills/SKILLS-ROADMAP.md`
**Review question:** For each skill: did any audit finding expose a gap the skill should have caught/prevented? If so, what update closes the gap?

| Skill | Current Version | Relevant Findings | Gap Identified | Proposed Update | Tier | Target Session |
|---|---|---|---|---|---|---|
| `{skill-name}` | v{x.y} | {Phase N F#, ...} | {one-line gap} | {one-line update} | {1/2/3} | {this/next} |
| `{skill-name}` | v{x.y} | — | none (scanned) | none | — | — |

**Skills with no gaps identified this quarter:**
- `{skill-name}`, `{skill-name}`, `{skill-name}` — scanned, no finding implicates them.

**Cross-skill observations:**
- {pattern spanning multiple skills, e.g., "4 COO skills reference X path; X was archived S{N}"}

---

### 2.2 CC Skills + Rules + Hooks

**Inventory source:** `AOS/CC-Config/MANIFEST.md`
**Review question:** For each CC component: did any finding expose a gap? Specific to CC: did any storm / compaction / silent-drop event suggest a rule or hook needs adjustment?

| Component | Type | Current Version | Relevant Findings | Gap Identified | Proposed Update | Tier |
|---|---|---|---|---|---|---|
| `{component}` | {skill/rule/hook/agent} | v{x.y} | {F#} | {gap} | {update} | {1/2/3} |

**Rule + hook specific:** For any finding that represents a CC mistake (silent-drop, wrong-doc edit, skipped verification): is this preventable at the rule/hook level? If yes, propose concrete rule addition OR hook implementation.

| Finding | Preventable via | Rule/Hook Proposed | Tier |
|---|---|---|---|
| {F#} | {rule / hook / skill} | {proposed change} | {1/2/3} |

---

### 2.3 SSOT-REGISTRY Cascade Rules

**Inventory source:** `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md §Cascade Update Rules`
**Review question:** Does Phase 7 §3 Patterns surface new cascade pairs that need explicit rules? Are existing rules still accurate?

**Existing Cascade Rules review:**

| Rule # | Pair | Still Valid? | Changes Needed |
|---|---|---|---|
| #{N} | {A → B} | {yes/no} | {one-line} |

**New Cascade Rules proposed (from this audit's Patterns):**

| Proposed Rule | Source Pattern | Leverage | Tier |
|---|---|---|---|
| {A → B: describe trigger + cascade} | Pattern {X} | {L/M/H} | {1/2/3} |

---

### 2.4 DOC-GOVERNANCE Routing Precedents

**Inventory source:** `HAIOS/Knowledge/DOC-GOVERNANCE.md §Routing Precedents`
**Review question:** Phase 7 §3 Patterns labeled "proposed Routing Precedent" — formalize them here. Also: do any existing precedents need clarification / extension based on findings?

**New Precedents proposed (formalize from Phase 7 §3):**

| Precedent (imperative) | Source Pattern | Phase 7 §3 label | Tier |
|---|---|---|---|
| {"When X, do Y."} | Pattern {A/B/C} | ✅/⚠/⏳ | {1/2/3} |

**Existing Precedents review:**

| Precedent | Reviewed? | Changes Needed |
|---|---|---|
| {existing precedent} | yes | {one-line} |

---

### 2.5 Anti-Patterns + Watch-list

**Inventory source:** `AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md`
**Review question:** Watch-list entries ready for promotion to Active? New APs surfaced? Existing APs superseded?

**Watch → Active promotions:**

| Watch # | Proposed anti-pattern | Evidence count since added | Rationale for promotion | Tier |
|---|---|---|---|---|
| W{N} | anti-pattern {N} | {N} | {one-line} | {1/2/3} |

**New APs (not in Watch-list yet):**

| Proposed anti-pattern | Finding Source | Evidence | Tier |
|---|---|---|---|
| anti-pattern {N} | {F#} | {one-line} | {1/2/3} |

**AP retirements (no evidence in last 3+ audits):**

| anti-pattern | Last Evidence | Rationale for Retirement |
|---|---|---|
| anti-pattern {N} | S{N} | {one-line} |

---

### 2.6 Tools / n8n Workflows

**Inventory source:** `AOS/Workflows/` + `AOS/Domains/SERVICE-CATALOG.md`
**Review question:** Which findings could have been prevented or detected by automation (a hook, an n8n workflow, a scheduled scan)? What's the build cost vs. audit-cycle cost?

| Finding / Pattern | Automation Proposed | Estimated Build Cost | Routes to | Tier |
|---|---|---|---|---|
| {F# or Pattern} | {hook / n8n workflow / script} | {S or $} | {ROADMAP target} | {1/2/3} |

---

## 3. Tier 1 Standalone Proposals (queued for NEXT session)

**Routing:** These do NOT append to the Phase 6 decision memo for this quarter. They are staged in `session-context.md` NEXT SESSION as standalone Tier 1 items, resolved in the next session after the audit closes.

Each Tier 1 item here uses the same entry template as `PHASE-6-DECISION-MEMO-TEMPLATE.md` single-decision format — but owned by Phase 8, not Phase 6.

### P8-T1-{N}: {title}

**Source:** Phase 8 §2.{subsection} review
**Underlying findings:** {Phase N F#, ...}

**What's proposed:** {1-2 sentences. Concrete component + file path + specific change.}

**Evidence:** {1-2 sentences referencing review row + underlying findings.}

**COO recommendation:** {APPROVE | MODIFY | REJECT} — {one-line reason}

**Alternatives considered:**
- **{Alt A}:** {one-line reason not preferred}
- **{Alt B}:** {one-line reason not preferred}

**Impact if approved:** {Downstream effects; who executes; what cascades.}
**Impact if rejected:** {Status quo consequences.}

**Resolution needed by:** {next session | end of {month} | specify date}

---

## 4. Tier 2/3 Executions This Session

Changes COO executed autonomously during Phase 8. Each row cites before/after for traceability.

| # | Component | Change | Before | After | Finding Source |
|---|---|---|---|---|---|
| 1 | `{file/skill/rule}` | {summary} | v{x.y} | v{x.(y+1)} | {F#} |

**Version-bump summary:**

| Doc / skill / rule | From | To | Session |
|---|---|---|---|
| `{path}` | v{x.y} | v{x.(y+1)} | S{N} |

---

## 5. Cross-Quarter Pattern Observations

Findings this quarter that RECUR from past quarterly audits. Each row: what's recurring, which quarters have seen it, and whether current fixes are sufficient. If recurrence continues despite prior fixes → structural issue warranting Tier 1 escalation.

| Pattern | Quarters seen | Prior fix attempts | Still recurring? | Escalation |
|---|---|---|---|---|
| {one-line} | Q{N}, Q{N+1}, ... | {S{N} fix, ...} | {yes/no} | {Tier 1 proposal #N / resolved} |

---

## 6. Methodology Retrospective

Narrow scope: what worked / what to change for the `quarterly-doc-audit-skill` itself this quarter. Broader systemic observations live in §2-§5.

Candidate improvements surfaced:

- {one-line improvement} → codify in skill v{x.y} → {target quarter for change}
- {...}

**Skill version bump from this audit:** `quarterly-doc-audit-skill` v{x.y} → v{x.(y+1)} if applicable. Detail in its §Changelog.

---

## Changelog

| Version | Session | Changes |
|---|---|---|
| {x.y} | S{N} | {summary} |

````

---

## Template vs. actual review: what changes per quarter

| Stable across quarters | Per-quarter |
|---|---|
| Component-type subsections (6) | Which rows populate in each |
| Review row templates per subsection | Specific findings, specific components |
| Tier 1 routing (standalone, not Phase 6) | Tier 1 proposal count |
| Tier 2/3 execution table schema | Specific changes executed |
| Cross-quarter pattern logic | Which patterns have historical data |

Do not change subsection numbering (§2.1-§2.6) across quarters — external refs depend on it.

---

## Integration with session-context.md

After Phase 8 completes, the COO running `session-end-skill` stages each `P8-T1-{N}` item under `session-context.md` NEXT SESSION as:

```markdown
- [O] **Phase 8 Tier 1 #{N} — {title}.** From `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md` §3 P8-T1-{N}. COO recommends: {APPROVE | MODIFY | REJECT}. Resolution needed by: {date}.
```

NEXT SESSION entries persist until resolved. Once Jordan resolves in the next session, `session-end-skill` moves them to RECENT SESSIONS with outcome.

---

## Changelog of this template

| Version | Session | Changes |
|---|---|---|
| 1.0 | 592 | Initial creation. Derived from Q2-2026 audit directive to systematize feedback-loop-closing. Codifies 6-component review structure (COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade, DOC-GOVERNANCE precedents, Anti-Patterns, tools/n8n), Tier 1 standalone routing (not Phase 6 memo), Tier 2/3 auto-execution guidance, staleness pre-check, cross-quarter pattern tracking. |
