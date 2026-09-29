# Phase 6 — Jordan Decision Memo Template

**Used by:** `quarterly-doc-audit-skill` Step 4 (COO + Jordan Review)
**When:** After Phases 1-5 complete and all Tier 1 findings are aggregated
**Purpose:** Consistent format across audits so Jordan can process Tier 1 decisions efficiently. Each entry readable in ~2 minutes.

---

## Format rules

- **Present in chat.** Decisions happen live in the COO chat session. Persist resolutions to `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}.md` §5 Decision Log as they resolve.
- **One decision per entry.** Exception: similar decisions are BATCHED per the Batching rules below.
- **Target ≤50 lines per entry in chat.** Longer than that = Jordan won't process in one read.
- **Use the entry template verbatim.** Deviations cost Jordan reading time and break the scan pattern.
- **Order by impact first, Tier second.** Top-3 most-impactful findings lead; simpler decisions follow. Jordan's attention is highest at the start of a decision block.

---

## Single-decision entry template

```markdown
### Decision {N} — {one-line title}

**Source:** Phase {N} Finding {F#} | **Severity:** {score} | **Type:** {classification}

**What's proposed:** {1-2 sentences. Concrete — file path, specific value, named section.}

**Evidence:** {Line/section refs from Phase output. 1-2 sentences max. Link to source phase report.}

**COO recommendation:** {APPROVE | MODIFY | REJECT} — {one-line reason}

**Alternatives considered:**
- **{Alternative A}:** {1-line reason not preferred}
- **{Alternative B}:** {1-line reason not preferred}

**Impact if approved:** {Downstream effects. Who executes, when, what cascades.}
**Impact if rejected:** {Status quo consequences. What drifts, what recurs.}

**Resolution needed by:** {within-session | end-of-session | end-of-week | specify date}

---
```

### Example — single decision

```markdown
### Decision 1 — `README-PROTOCOL.md` zombie deprecation

**Source:** Phase 2 Finding F10 | **Severity:** Reference conflict | **Type:** Circular SSOT

**What's proposed:** Archive `HAIOS/Knowledge/README-PROTOCOL.md` (1-line stub: "Superseded by claude-instructions.md §Secrets. Archived.") + strip 2 active callers (`HAIOS/claude-instructions.md` lines 128 and 163) that still cite the stub as authoritative protocol.

**Evidence:** Phase 2 report F10 + Phase 1 inventory Notes. Stub marked "archived" in body but file never moved; 2 callers treat it as canonical. True circular SSOT.

**COO recommendation:** APPROVE — content already migrated to `claude-instructions.md` §Secrets; stub adds no value and misdirects readers.

**Alternatives considered:**
- **Restore stub to full content:** content already lives elsewhere; would duplicate canonical source.
- **Leave stub + update callers only:** leaves 1-line zombie on disk; next audit surfaces it again.

**Impact if approved:** `README-PROTOCOL.md` → `Archive/HAIOS/`. `claude-instructions.md` lines 128/163 rewritten to inline or reference current Secrets Protocol. Archive-move sweep (session-end-skill v1.7) verifies no other callers exist.
**Impact if rejected:** Circular SSOT persists. Every reader hitting line 128/163 gets routed to a dead stub. Next audit re-surfaces.

**Resolution needed by:** end-of-session (Phase 5 COMPLIANCE may need to know whether stub is in scope).

---
```

---

## Batching rules

When multiple findings share a pattern (e.g., 19 docs all missing the same frontmatter field), batch them into ONE decision entry. Rules:

1. **Same root cause, same fix pattern.** If the proposed action is identical per-finding modulo the target file, batch.
2. **One Jordan decision applies to all.** Don't present per-finding decisions for pattern items.
3. **Present pattern + sample + full-list link.** Sample of 3 is enough context; don't dump all N.
4. **Execution is per-instance but decision is patternwide.** Once pattern approved, each instance executes as a Tier 2 mechanical task.

**Never do this:** Present 19 separate decisions for the same pattern. That burns Jordan's capacity on repeat items and obscures the real decisions.

---

## Batched-decision entry template

```markdown
### Decision {N} — {pattern name} (BATCH: {count} findings)

**Source:** Phase {N} — pattern across {count} findings
**Scope:** {1 line on what the pattern is}

**Pattern description:** {2-3 sentences explaining the common thread.}

**Sample findings (3 of {count}):**
1. `{file path}` — {specific issue}
2. `{file path}` — {specific issue}
3. `{file path}` — {specific issue}

**Full affected list:** see Phase {N} report §{section} (N={count}).

**COO recommendation:** {APPROVE PATTERN | MODIFY PATTERN | REJECT} — {one-line reason}

**Proposed resolution pattern:** {1-2 sentences on the uniform action per-instance.}

**Execution plan if approved:**
- Tier 1 approval → pattern locked
- Tier 2 batch execution: {N findings × mechanical edit each; COO executes OR queues CC task}
- Verification: {spot-check method after batch completes}

**Impact if approved:** {pattern fix + prevention going forward}
**Impact if rejected:** {per-case manual handling, ongoing drift}

**Resolution needed by:** {timing}

---
```

### Example — batched decision

```markdown
### Decision 5 — 19-doc `ssot-for` frontmatter gap (BATCH: 19 findings)

**Source:** Phase 5 Pass 2 — pattern across 19 findings
**Scope:** 19 of 24 SSOT-Y docs in Phase 1 inventory lack `ssot-for:` frontmatter field

**Pattern description:** Phase 1 identified 24 docs marked SSOT-Y via registry membership. Only 5 have the `ssot-for:` field explicitly in frontmatter. The other 19 are SSOT-via-registry-only. Strict rule says "any `ssot-for:` addition = Tier 1" — but applying that to 19 separate findings burns decision-making capacity unproductively.

**Sample findings (3 of 19):**
1. `HAIOS/Architecture/AUTONOMOUS-EXECUTION-ARCHITECTURE.md` — SSOT-Y per registry; no `ssot-for:` field
2. `HAIOS/Knowledge/DOC-GOVERNANCE.md` — SSOT-Y per registry; no `ssot-for:` field
3. `HAIOS/Knowledge/BestPractices/CC-DEVELOPMENT-PLAYBOOK.md` — SSOT-Y per registry; no `ssot-for:` field

**Full affected list:** Phase 1 inventory Flagged section "Phase 5 pattern — SSOT=Y docs missing `ssot-for` frontmatter field (19 docs)".

**COO recommendation:** APPROVE PATTERN — purely additive. No content changes. Enables future bi-directional SSOT drift detection (file ↔ registry both self-describe).

**Proposed resolution pattern:** For each of the 19 docs: add `ssot-for: {registry-domain-slug}` to frontmatter. Value sourced per-doc from SSOT-REGISTRY-MASTER.md domain claim. Version bump on each doc (x.y → x.(y+1)).

**Execution plan if approved:**
- Tier 1 approval → pattern locked
- Phase 5.5 CC dispatch: mechanical batch edit. 19 docs, one registry lookup per doc, one frontmatter addition per doc.
- Verification: re-run Phase 5 Pass 1 Check B on the 19 docs; all should pass post-batch.

**Impact if approved:** Registry ↔ file bi-directional link becomes self-describing. Future quarterly audits catch drift faster (either direction fails independently).
**Impact if rejected:** Ongoing asymmetry — registry is SSOT for the relation but individual files can't self-identify. Bi-directional check impossible.

**Resolution needed by:** Phase 6 completion (before Phase 7 synthesis, so registry deltas stabilize).

---
```

---

## Presentation ordering in chat

When Jordan enters Phase 6, COO presents decisions in this order:

1. **Top-3 most-impactful** (per Phase 7 synthesis report §1) — even if low-Tier by classification, these deserve attention first because they shape the Phase 7 narrative.
2. **Tier 1 batched decisions** — pattern resolutions. One decision affects many findings; high leverage.
3. **Tier 1 individual decisions** — single findings requiring Jordan's call.
4. **Tier 1 items that surface new Routing Precedents for DOC-GOVERNANCE** — these inform future audits and deserve explicit discussion.

Within each bucket, order by reversibility: **irreversible decisions first** (deprecations, content deletions), **reversible decisions second** (moves, link adds), **pure additions last** (new frontmatter fields, new registry rows).

Rationale: Jordan's concentration is highest at the start. Irreversible decisions deserve that concentration.

---

## Closing — decision roll-up

After Jordan resolves all Tier 1 decisions, COO:

1. Populates Phase 7 report §5 Decision Log table. Each row: `# | Source | Decision | COO Recommendation | Jordan Resolution`.
2. Updates the §6 Execution Plan to reflect which decisions unlock which executions.
3. Flags any decisions Jordan deferred to next quarter in §6 "Deferred to Q{N+1}-{YEAR}" subsection.

Tier 2 batched executions (e.g., the 19-doc `ssot-for` batch) can proceed autonomously once Tier 1 pattern approval lands — COO may dispatch the execution CC task in the same session as Phase 6 or queue for a later session.

---

## Changelog

| Version | Session | Changes |
|---|---|---|
| 1.0 | 591 | Initial creation. Derived from Q2-2026 audit execution. Codifies single-decision + batched-decision entry formats, presentation ordering rules, decision roll-up protocol. |
