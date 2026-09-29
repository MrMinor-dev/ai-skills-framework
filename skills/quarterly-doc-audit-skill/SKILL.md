---
name: quarterly-doc-audit-skill
description: "Run the quarterly L2 doc audit pass (Jan/Apr/Jul/Oct). Use when: quarterly doc audit, Q[N] [YEAR] doc audit, run doc audit, doc audit [HAIOS|AOS], post-tech-debt conflict inspection, L2 doc audit."
---

# Quarterly Doc Audit

**Version:** 2.5 | **Layer:** AOS | **Created:**

Recurring quarterly L2 judgment pass on doc trees per `HAIOS/Knowledge/DOC-GOVERNANCE.md` §Operational Playbook. Each audit scopes to a target doc area (HAIOS, AOS, etc.), runs **8 phases** (inventory → conflict → staleness → consolidation → compliance → decisions → synthesis → systemic review), produces a synthesis report + systemic-improvement review + doc-governance feedback cascade.

## Context

Quarterly L2 audits are the judgment-pass cadence mandated by DOC-GOVERNANCE. L1 runs weekly (automated detection via `HAIOS-Infra-Doc-Governance` workflow); L2 runs quarterly (judgment-heavy structural review). Anchors: January, April, July, October.

Each quarter produces these permanent artifacts + ephemeral dispatch prompts. **Path base:** `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/` — note the literal folder name, spaces and `[L2]` included.
- **Scope doc** at `{base}/DOC-AUDIT-Q{N}-{YEAR}-SCOPE.md` — defines what's in scope for THAT quarter + known hotspot pairs + Open Questions for Jordan.
- **Execution prompts** at `{base}/CC-PROMPT-AUDIT-Q{N}-{YEAR}-PHASE-{N}-{TOPIC}.md` — per-phase CC prompts (ephemeral, archived post-run).
- **Phase outputs** at `{base}/DOC-AUDIT-Q{N}-{YEAR}-PHASE-{1-5}-*.md` — CC outputs for Phases 1-5 (permanent, traceability).
- **Phase 6 decision memo** at `{base}/DOC-AUDIT-Q{N}-{YEAR}-PHASE-6-DECISION-MEMO.md` — Jordan-facing Tier 1 decisions (permanent).
- **Synthesis report** at `{base}/DOC-AUDIT-Q{N}-{YEAR}.md` — Phase 7 synthesis output (permanent, primary deliverable).
- **Systemic review** at `{base}/DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md` — Phase 8 component-by-component improvement analysis (permanent).

The 8-phase methodology is stable across quarters; the scope target rotates (see `## Scope Rotation`).

**Why 8 phases (added):** Phases 1-5 detect findings. Phase 6 resolves Tier 1 decisions. Phase 7 synthesizes. **Phase 8 closes the feedback loop** — translates findings into concrete improvements to COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade rules, DOC-GOVERNANCE routing precedents, Anti-Patterns, and automation tools. Without Phase 8, the audit reports drift but doesn't systematically fix the system that allowed drift.

## Workflow

### 1. GATHER KNOWLEDGE

Read the per-quarter scope doc:
- `Filesystem:read_file` → `AOS\Workflows\HAIOS - Infra - Doc Governance [L2]\DOC-AUDIT-Q{N}-{YEAR}\DOC-AUDIT-Q{N}-{YEAR}-SCOPE.md`

If no scope doc exists for the current quarter, create one first. Use the previous quarter's scope doc as template; update `triggers`, `scope_in` / `scope_out`, and known hotspot pairs based on structural events since the last audit.

Also read (reference methodology):
- `HAIOS/Knowledge/DOC-GOVERNANCE.md` §Operational Playbook (parent methodology)
- `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-PLAN.md` (baseline — general audit structure)
- `Archive/DOC-AUDIT-PATTERNS-CAPTURE.md` (prior 8-phase worked output as structural precedent — archived, read-only reference)

### 2. CONFIRM SCOPE + OPEN QUESTIONS

Surface all Open Questions from §7 of the scope doc with COO defaults + COO recommendation per item. Present as a decision table to Jordan. Do NOT dispatch CC until scope is locked. Tier 1 decisions only — no autonomous override.

### 3. DISPATCH PHASES 1-5 TO CC

Each phase is its own CC prompt file. Use `cc-prompt-skill` AUDIT task type. Read `references/task-types/AUDIT.md` session learnings before drafting each prompt. Phase-specific prompt templates live in this skill's `references/`.

| Phase | Scope | Template | Output |
|---|---|---|---|
| **1 — Inventory** | List every `.md` file in scope_in with size, last-update, category, SSOT flag | `references/PHASE-1-INVENTORY-PROMPT.md` | `DOC-AUDIT-Q{N}-{YEAR}-PHASE-1-INVENTORY.md` |
| **2 — Conflict Detection** | Pairwise cross-doc comparison; parallel subagents one per hotspot pair + one sweep | `references/PHASE-2-CONFLICT-PROMPT.md` | `DOC-AUDIT-Q{N}-{YEAR}-PHASE-2-CONFLICTS.md` |
| **3 — Staleness** | Per-doc freshness, broken refs, deprecated doctrine, summary/body drift | `references/PHASE-3-STALENESS-PROMPT.md` | `DOC-AUDIT-Q{N}-{YEAR}-PHASE-3-STALENESS.md` |
| **4 — Consolidation** | Topic-level fragmentation, promotion candidates | `references/PHASE-4-CONSOLIDATION-PROMPT.md` | `DOC-AUDIT-Q{N}-{YEAR}-PHASE-4-CONSOLIDATION.md` |
| **5 — Compliance** | Frontmatter + SSOT-REGISTRY walk + path verification | `references/PHASE-5-COMPLIANCE-PROMPT.md` | `DOC-AUDIT-Q{N}-{YEAR}-PHASE-5-COMPLIANCE.md` |

**Dispatch cadence:**
- **Phase 1 first** — blocks all others. Single CC session, Sonnet, medium effort.
- **Phases 2, 3, 4, 5 run in parallel** after Phase 1 completes. Each is its own CC session; resource conflict check before dispatch (all READ Phase 1 output + scope files; WRITES are distinct per-phase output files → safe to parallelize).
- Total budget: 3-5 hrs across many sessions. = Phase 1 dispatch + absorb + Phases 2-5 dispatch + absorb. = Phases 6-8.

**Per-quarter prompt instantiation:** Copy the template from `references/`, fill in quarter-specific placeholders (`{N}`, `{YEAR}`, scope_in paths, hotspot pairs from scope doc §2, open-question resolutions). Write to `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/CC-PROMPT-AUDIT-Q{N}-{YEAR}-PHASE-{N}-{TOPIC}.md`. Jordan kicks off with standard one-liner: `Read the task prompt at "{filepath}" — use Plan Mode, then execute.`

**Phase 4 dispatch notes:**
- **Subagent local filesystem:** Add to the top of EACH Phase 4 subagent prompt: "IMPORTANT: All files below are on the LOCAL Windows filesystem. Google Drive is mounted locally. Use the Read tool directly — do NOT attempt any MCP authentication." Without this, subagents may request MCP auth for `` paths non-deterministically (Note: 3 of 6 affected; retry resolved). See Watch W53 in ANTI-PATTERNS.md.
- **Phase 3/Phase 4 finding overlap:** If a Phase 4 finding overlaps with a Phase 3 broken-ref finding (same doc/reference, already tracked for Phase 5 path update), include it only if Phase 4 analysis adds a distinct dimension (e.g., cross-reference quality or authority gap vs. pure path update). Note the overlap with a Phase 5 lane reference so the report reader can disambiguate.

### 4. COO + JORDAN REVIEW (Phase 6)

COO reads all Phase 1-5 outputs + feedback files. Aggregate findings into a Tier-sorted decision list.

Tier definitions follow `HAIOS/CEO-COO-CONTRACT.md §Part 6 Authority Tiers` (SSOT). Audit-specific mapping of tiers to finding classes:

- **Tier 1** (Jordan approval): deprecations, SSOT reshuffles, canonical rehoming, authority-level changes. Example findings: move doc X from Knowledge to Architecture; deprecate doc Y to Archive; rename SSOT claim on doc Z.
- **Tier 2** (inform after): link additions, frontmatter fixes (>10 batch), stale-ref strips with clear targets. Example findings: add cross-ref from A to B; strip 12 broken paths; bump 8 stale `updated` fields.
- **Tier 3** (batch-executed): grammar, format, verified-broken-link fixes, missing `updated` field. Already executed during Phase 5 pass; log for traceability only.

Present Tier 1 decisions to Jordan in decision-memo format using **`references/PHASE-6-DECISION-MEMO-TEMPLATE.md`** (codifies single-decision + batched-decision entry formats, presentation ordering, and roll-up protocol). Jordan picks lane per finding. Same-day triage rule: SSOT conflicts on active-use docs or broken refs on docs read every session → triage inline, don't wait for full synthesis.

### 5. SYNTHESIS REPORT (Phase 7)

COO writes the audit synthesis report at `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/DOC-AUDIT-Q{N}-{YEAR}.md` using **`references/PHASE-7-SYNTHESIS-REPORT-TEMPLATE.md`** as the structural template (codifies §1-§7 layout, populated-vs-stub markers, and Phase 8 forward reference). Use `doc-management-skill` for creation (structured multi-section response, Persist-Before-Present).

Required structure (per template):

1. **Executive summary** — headline counts + top-3 impactful findings + preventions-applied-in-session log
2. **Action table** — per finding: severity | proposed fix | owner | tier | blocker-y/n
3. **Pattern observations** — recurring themes; feeds Phase 8 DOC-GOVERNANCE review
4. **SSOT-REGISTRY-MASTER deltas** — bumps in-session, additions proposed, removals/supersessions
5. **Decision log** — Tier 1 resolutions from Phase 6 (Jordan's calls + COO recommendations)
6. **Execution plan** — what's in-session, what's queued as separate CC tasks, what's pending Jordan
7. **Cascade updates post-report** — scoped to THIS audit's findings (DOC-GOVERNANCE, SSOT-REGISTRY, session-context FLAGS). Systemic-level improvements (skill changes, new rules) migrate to Phase 8.

**Methodology retrospective migrated to Phase 8 starting v2.0.** Phase 7 focuses on this audit's findings report; Phase 8 covers "what does this tell us about our system?" Section numbering is load-bearing — Phase 8 references Phase 7 by section number, so do not renumber across quarters.

### 6. SYSTEMIC REVIEW (Phase 8)

**Purpose:** Translate audit findings into concrete improvement proposals for COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade rules, DOC-GOVERNANCE routing precedents, Anti-Patterns, and automation tools. Closes the feedback loop from findings back into the system that generated them.

**Execution model:** COO runs directly (Opus). Cross-system reasoning over the entire MRMINOR architecture — not a CC-dispatchable task. Subagents may assist on component inventory gathering; synthesis stays in Opus main context.

**Input:** Phase 7 synthesis report (primary) + Phase 1-5 reports (detail sources as needed).

**Output:** `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md` per **`references/PHASE-8-SYSTEMIC-REVIEW-TEMPLATE.md`**.

**Review structure (per component type):**

| Component Type | Inventory Source | Review Question |
|---|---|---|
| COO Skills | `AOS/Skills/SKILLS-ROADMAP.md` | What finding should this skill have caught / prevented? What update closes that gap? |
| CC Skills + Rules + Hooks | `AOS/CC-Config/MANIFEST.md` | Same — per skill/rule/hook |
| SSOT-REGISTRY Cascade Rules | `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md` §Cascade Update Rules | Do Phase 7 patterns suggest new rules? |
| DOC-GOVERNANCE Routing Precedents | `HAIOS/Knowledge/DOC-GOVERNANCE.md` §Routing Precedents | Formalize Phase 7 §3 patterns as new precedents |
| Anti-Patterns + Watch-list | `AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md` | New entries, Watch→Active promotions, retirements |
| Tools / n8n Workflows | `AOS/Workflows/` + `AOS/Domains/SERVICE-CATALOG.md` | Which findings are automatable as detection workflows or hooks? |

**Output structure:**
- §1 Executive Summary (counts across component types)
- §2 Per-Component Review (one subsection per component type above)
- §3 Tier 1 Standalone Proposals (queued for NEXT session — NOT appended to Phase 6 memo)
- §4 Tier 2/3 Executions This Session (auto-executed with before/after refs)
- §5 Cross-Quarter Pattern Observations (recurring structural issues)
- §6 Methodology Retrospective (narrow: audit skill evolution)

**Staleness pre-check:** If `SKILLS-ROADMAP.md` or `CC-Config/MANIFEST.md` is stale relative to the latest session, REFRESH them FIRST. A stale inventory invalidates Phase 8's coverage claim.

**Timing:** Executes AFTER Phase 7 synthesis closes (needs Phase 7 as input). Becomes the final act of the audit cycle.

### 7. CASCADE

After Phase 7 synthesis + Phase 8 systemic review both complete, execute the feedback loops:

**From Phase 7 (audit-specific findings):**
- **`HAIOS/Knowledge/DOC-GOVERNANCE.md` Routing Precedents** — append novel-case decisions from this audit. Bump version.
- **`HAIOS/Architecture/SSOT-REGISTRY-MASTER.md`** — apply Phase 5 deltas + any Phase 6 approved registry additions. Bump version.
- **`HAIOS/session-context.md`** — update FLAGS, remove resolved triggers, add FLAG for next quarter's audit anchor.

**From Phase 8 (systemic improvements):**
- Execute Phase 8 Tier 2/3 proposals in-session (skill version bumps, cascade rule additions, Anti-Pattern promotions).
- Stage Phase 8 Tier 1 proposals in `session-context.md` NEXT SESSION as standalone decision items for next session (do NOT append to Phase 6 decision memo).
- `AOS/Skills/SKILLS-ROADMAP.md` — update entries for any skills modified this audit cycle.
- `AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md` — add new APs or Watch-list entries surfaced during prompt drafting / CC feedback.
- **New L1 detection patterns** — if Phase 8 identifies automation opportunities, queue build items in appropriate `*-BUILD-ROADMAP.md` → `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/` scope.

### 8. ARCHIVE

Move per-quarter ephemeral artifacts to the ONE flat `MRMINOR/Archive` with self-locating names (per `doc-management-skill` §Session artifact routing + `HAIOS/Knowledge/ARCHIVE-ARCHITECTURE-DECISION.md`). Never to `Archive/CC-Prompts/` or any range subfolder — that convention was retired.
- `CC-PROMPT-AUDIT-Q{N}-{YEAR}-PHASE-*.md`
- `CC-FEEDBACK-AUDIT-*.md`
- `CC-BUILD-LOG-AUDIT-*.md`

Keep at `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/` (permanent historical, all in subfolder):
- `DOC-AUDIT-Q{N}-{YEAR}-SCOPE.md`
- `DOC-AUDIT-Q{N}-{YEAR}.md` (Phase 7 synthesis report)
- `DOC-AUDIT-Q{N}-{YEAR}-PHASE-1-INVENTORY.md` through `PHASE-5-COMPLIANCE.md` (CC outputs)
- `DOC-AUDIT-Q{N}-{YEAR}-PHASE-6-DECISION-MEMO.md` (Jordan-facing decisions)
- `DOC-AUDIT-Q{N}-{YEAR}-PHASE-8-SYSTEMIC-REVIEW.md` (Phase 8 report)

Verify no orphans per `cc-prompt-skill` §5 archive gate.

## Handoff Contract

When audit concludes, state:
- Audit period: Q{N} {YEAR}
- Scope: {HAIOS | AOS | both}
- Deliverables: synthesis report path, systemic review path, scope doc path, retained phase outputs
- Findings headline: {N conflicts, N stale, N consolidation, N compliance, N deprecation candidates}
- Tier 1 decisions resolved (Phase 6): {count} (detail in report §5)
- Systemic improvements (Phase 8): {N Tier 2/3 executed this session, M Tier 1 queued for next session}
- Cascade complete: DOC-GOVERNANCE / SSOT-REGISTRY / ANTI-PATTERNS / SKILLS-ROADMAP / session-context FLAGS all updated
- Next audit scheduled: {quarter} {year} — FLAG set in session-context

## Dependencies

- Filesystem tools (read scope, write report, archive via moves)
- `cc-prompt-skill` (drafting per-phase AUDIT prompts)
- `semantic-search-skill` (CC CONTEXT QUERIES + COO context gathering during Phases 6, 8) **— invoke explicitly. Q2-2026 audit closed without COO invoking semantic search despite it being listed here (post-close finding). Phase 6: run 2-3 queries to surface cross-doc patterns before aggregating findings. Phase 8: run component inventory cross-checks before writing §2 subsections.**
- `doc-management-skill` (Phase 7 + 8 report creation — structured multi-section)
- `HAIOS/Knowledge/DOC-GOVERNANCE.md` (parent methodology)
- Per-quarter scope doc at `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/DOC-AUDIT-Q{N}-{YEAR}/DOC-AUDIT-Q{N}-{YEAR}-SCOPE.md` (created before dispatch)

## References

- `references/PHASE-1-INVENTORY-PROMPT.md` — Phase 1 CC prompt template
- `references/PHASE-2-CONFLICT-PROMPT.md` — Phase 2 CC prompt template
- `references/PHASE-3-STALENESS-PROMPT.md` — Phase 3 CC prompt template
- `references/PHASE-4-CONSOLIDATION-PROMPT.md` — Phase 4 CC prompt template
- `references/PHASE-5-COMPLIANCE-PROMPT.md` — Phase 5 CC prompt template
- `references/PHASE-6-DECISION-MEMO-TEMPLATE.md` — Phase 6 Jordan-facing decision memo format (v1.0)
- `references/PHASE-7-SYNTHESIS-REPORT-TEMPLATE.md` — Phase 7 synthesis report structure (v1.0)
- `references/PHASE-8-SYSTEMIC-REVIEW-TEMPLATE.md` — Phase 8 systemic review output structure (v1.0)

## Authority

- Audit execution (drafting prompts, dispatching CC, absorbing results, Phase 7 synthesis, Phase 8 review): **Tier 3** (autonomous)
- Tier 2 batch fixes post-report (link adds, frontmatter fixes, stale-ref strips, version bumps, AP promotions, cascade rule additions): **Tier 2** (inform after)
- **Tier 1 audit findings** (deprecations, SSOT reshuffles, canonical rehoming, authority-level changes): **Tier 1** (Jordan approval, Phase 6 memo)
- **Tier 1 Phase 8 proposals** (new skills, new rules, structural hooks, breaking changes to components): **Tier 1** (Jordan approval, queued for NEXT session via standalone §3)
- DOC-GOVERNANCE Routing Precedents additions: **Tier 2** (inform after)

## Cadence

Target execution windows:
- Q1 audit: January
- Q2 audit: April
- Q3 audit: July
- Q4 audit: October

Each audit must conclude before the next quarter's anchor. When an audit concludes, set a FLAG in `session-context.md` for the next anchor + scope target. Missed quarters cascade — if Q2 slips into May, Q3's window compresses. Don't double-up.

## Scope Rotation

Scope target rotates to prevent over-auditing any single tree:

- **Q2-2026:** HAIOS (Architecture + Knowledge + Best Practices) ← current
- **Q3-2026:** AOS (planned — Architecture + Strategy + Domains + Skills)
- **Q4-2026:** TBD per structural events in the intervening months; likely HAIOS if AGENCY workstream grows, or AOS if workflow count expands significantly.
- **Q1-2027:** rotate back to opposite tree of Q4-2026.

Do not audit both trees in one quarter unless (a) a cross-layer conflict investigation is the trigger, or (b) trees are unusually small. Quarterly budget targets 3-5 hrs; doubling scope doubles cost and risks compaction storms.

## Error Handling

- **Scope doc missing:** Create one using prior quarter's as template. Surface triggers to Jordan before writing.
- **CC Phase output incomplete (compaction mid-run):** Check `CC-BUILD-LOG` for pre-compaction progress. Re-dispatch with narrower scope if >50% complete; restart with subagent fan-out if <50%.
- **Hotspot pair doesn't exist anymore** (doc deleted between scope authoring and dispatch): verify via `Filesystem:search_files` in Phase 1 inventory absorption; drop the pair and note in Phase 2 prompt's IMPORTANT NOTES.
- **SSOT-REGISTRY walk returns inconsistency (Phase 5):** Do NOT auto-fix. Flag as Tier 1 for Jordan — registry is governance-tier.
- **Storm flag raised during CC phase:** Follow `cc-audit-skill` protocol immediately. Pause audit dispatch until storm resolved.
- **Phase 8 component inventory stale:** If `SKILLS-ROADMAP.md` or `CC-Config/MANIFEST.md` is out of date at Phase 8 start, refresh them FIRST. Stale inventory invalidates Phase 8's coverage claim.
- **Scope doc missing adjacent-candidate SSOT homes section** (v2.2): before any Pattern C promotion decision (3+ doc duplication → SSOT designation), scope doc must include an "Adjacent Candidate SSOT Homes" section listing docs one directory level above/below the audit scope that could be existing richest-home candidates. Q2-2026 Pattern C reversal traced to scope excluding HAIOS/ root, which missed `CEO-COO-CONTRACT.md §Part 6` — an already-fully-developed SSOT. If scope doc lacks this section at Phase 1 start, add it before Phase 4 consolidation runs.

## Cross-References

- Parent methodology: `HAIOS/Knowledge/DOC-GOVERNANCE.md` §Operational Playbook
- Prior precedent: `AOS/Workflows/HAIOS-Infra-Doc-Governance/DOC-AUDIT-PLAN.md` + `HAIOS/Architecture/DOC-AUDIT-PATTERNS-CAPTURE.md`
- Related forensic family: `AOS/Workflows/HAIOS-Intel-Analyzer/SILENT-DROP-REGRESSION-AUDIT.md`
- Related skill: `cc-prompt-skill` (for AUDIT task type)
- Related skill: `doc-management-skill` (for Phase 7 + 8 synthesis)
- Related skill: `cc-audit-skill` (for storm response during CC phases)
- Supersedes: archived draft `doc-audit-skill` (never formalized — stored at `Archive/Skills/doc-audit-skill.md`)

## Changelog

| Version | Session | Changes |
|---|---|---|
| 2.5 | 925 | **Stale-path repair — every artifact path in the skill pointed at a tree the artifacts don't live in.** v2.4 set the path base to `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}/`; the artifacts were subsequently relocated to `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/` and the skill never followed. Verified via `Filesystem:search_files`: all 10 Q2-2026 artifacts + the Q3-2026 scope doc live under the AOS/Workflows base; **zero** exist under `HAIOS/Architecture/`. Corrected: §Context artifact list (6 paths, now expressed against a stated `{base}`), §1 GATHER read path, §1 DOC-AUDIT-PLAN.md path (folder name was `HAIOS-Infra-Doc-Governance`, actual is `HAIOS - Infra - Doc Governance [L2]` — spaces + bracket suffix are literal), §1 DOC-AUDIT-PATTERNS-CAPTURE.md (now in `Archive/`, not `HAIOS/Architecture/`), §3 prompt instantiation path, §5 synthesis output, §6 Phase 8 output, §7 L1 scope pointer, §8 Keep-at header, §Dependencies scope path. **Also fixed §8 archive target:** pointed at `Archive/CC-Prompts/S{session}-*`, a convention retired in favour of the one flat `MRMINOR/Archive` — now aligned to `doc-management-skill` §Session artifact routing. Left untouched: the `SKILLS-ROADMAP.md` home conflict (`AOS/Skills/` vs `HAIOS/Architecture/`) — that is a live Q3-2026 Tier 1 decision per the Q3 scope doc, not a stale ref to silently resolve. Trigger: Jordan standing instruction — fix stale references on sight. |
| 2.4 | 596 | **Subfolder path template update.** All per-quarter artifact paths updated from flat `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}*.md` to `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}/DOC-AUDIT-Q{N}-{YEAR}*.md` subfolder pattern. Derived from that session Q2-2026 post-audit reorganization (9 files moved into subfolder to prevent loose files in Architecture root). Updated: §Context artifact list (6 paths), §1 GATHER KNOWLEDGE read path, §5 synthesis output path, §6 Phase 8 output path, §8 ARCHIVE "Keep at" header, §Dependencies scope doc path. |
| 2.3 | 596 | **Semantic search enforcement note.** Q2-2026 audit closed without COO invoking `semantic-search-skill` despite it being listed in §Dependencies (post-close finding). Added explicit "invoke explicitly" callout with Phase 6 and Phase 8 use cases to §Dependencies entry. |
| 2.2 | 595 | **Q2-2026 Phase 8 retrospective — 5 methodology improvements rolled up.** (1) **Pre-drafting Phase 2-5 prompts while Phase 1 runs** saved wall-clock time but required DRAFT banner + post-Phase-1 revision pass; codified as standard workflow practice. Phase 2-5 CC prompt drafts marked `DRAFT — awaiting Phase 1 absorption` until Phase 1 inventory lands. (2) **Phase 3 broken-ref subagent verification step explicit** (W52 pattern): Phase 3 CC prompt must include main-context Grep/Glob re-verification of all severity-1 broken-ref findings before inclusion in final report. Subagent negative-space assertions are not authoritative. (3) **Subagent MCP auth fallback note** (W53): every subagent prompt reading Drive files must include at TOP: "All files below are on LOCAL Windows filesystem. Google Drive is mounted locally. Use Read tool directly — do NOT attempt MCP authentication." Parent prompt header alone is insufficient. (4) **Phase 4 spot-check coverage proportional** (D7 phantom root cause): Phase 4 consolidation spot-check count must be `max(5, findings / 5)` — Q2-2026 Phase 4 used 3 fixed spot-checks for 35 findings; missed D7 phantom. (5) **Scope doc Adjacent Candidate SSOT Homes section** (Pattern C reversal root cause, added to Error Handling this version): before any Pattern C promotion decision, list docs one directory level above/below audit scope that could be existing richest-home candidates. **Pattern C default inverted** (already landed v2.1 re-referenced here for completeness). Phase 8 §6 Methodology Retrospective at `AOS/Workflows/HAIOS-Infra-Doc-Governance/DOC-AUDIT-Q2-2026-PHASE-8-SYSTEMIC-REVIEW.md` contains per-item evidence. Detailed phase-prompt text updates (items 2, 3, 4) stage for next-session Tier 2 CC dispatch; this version bump captures the intent and adds the scope-doc adjacent-candidates guidance inline. |
| 2.1 | 593 | §4 COO + Jordan Review: tier definition table stripped to link-to-SSOT (`CEO-COO-CONTRACT.md §Part 6`), retaining only audit-specific tier-to-finding-class mapping. Q2-2026 audit Decision 10 revision (Jordan: designate existing SSOT rather than create new BP doc). |
| 2.0 | 592 | **Structural:** Added Phase 8 (Systemic Review) as new final phase — closes the feedback loop from audit findings back into the system (COO skills, CC skills/rules/hooks, SSOT-REGISTRY cascade rules, DOC-GOVERNANCE routing precedents, Anti-Patterns, automation tools). Phase 7 scope narrowed (methodology retrospective migrated to Phase 8). Added `references/PHASE-7-SYNTHESIS-REPORT-TEMPLATE.md` (closes gap — primary deliverable previously had no template) and `references/PHASE-8-SYSTEMIC-REVIEW-TEMPLATE.md`. Workflow renumbered: Phase 8 inserted as step 6, Cascade → 7, Archive → 8. Authority section split for Phase 8 Tier 1 proposals (queue-for-next-session, not Phase 6 memo). Per-quarter artifacts list expanded. budget now covers Phases 6-8 (was 6-7). Derived from Q2-2026 audit directive to systematize feedback-loop-closing. |
| 1.2 | 591 | Added Phase 4 dispatch notes (§3): subagent local filesystem instruction (Watch W53) + Phase 3/Phase 4 finding overlap guidance. Derived from Q2-2026 Phase 4 feedback. |
| 1.1 | 591 | Added `references/PHASE-6-DECISION-MEMO-TEMPLATE.md` — codifies Phase 6 decision-memo format (single + batched), presentation ordering rules, decision roll-up protocol. Derived from Q2-2026 audit execution. Workflow Step 4 now references the template explicitly instead of deferring to `doc-management-skill`. |
| 1.0 | 590 | Initial creation. 7-phase methodology codified from Q2-2026 scope doc. Per-phase CC prompt templates queued for `references/` (Phase 1 template + live instantiation written Phases 2-5 templates written progressively as Q2-2026 executes). Cadence + scope rotation documented. Replaces archived draft `doc-audit-skill` (never formalized). |
