---
name: doc-management-skill
version: 2.23
updated: 2026-07-24
description: "Create/edit docs on Jordan's Drive. Triggers: any file write/edit in MRMINOR/; a doc Jordan asked for; a named output path; an ongoing artifact he will reopen. NOT triggered by output shape — a review, analysis, plan, or audit Jordan asked to SEE is a chat answer, not a file. Never create a file Jordan did not request (claude-instructions v25.40)."
---

# Document Management

Create and edit documents that agents can find, understand, and act on without human intervention.

> **Filing Mandate:** Before creating any new page, read `references/LAYER-TAXONOMY.md` and route by **primary subject** — not by source format, skill name, or where similar things already happen to live. The resolver fractal operates at the content layer too. (OPUS-4-7 tracker #27.)

## Quick Reference

Agentic docs require: semantic searchability (headers = concepts), unambiguous references (full paths always), progressive disclosure (lean core, detail on demand), SSOT integrity (one location per topic). Verify against agentic standards before completing.

## Workflow

### 0. PRE-FLIGHT (before ANY file operation)
- [ ] Target path starts with `<drive-root>/` — NOT `/home/claude/`, NOT `/mnt/`
- [ ] If modifying system component (skills, workflows, configs): semantic searched for related files/backups FIRST
- [ ] Existing file → `Filesystem:edit_file`. New file → `Filesystem:write_file`
- [ ] New file → searched to confirm no existing file covers this topic
- **New file → `references/LAYER-TAXONOMY.md` read THIS session before picking target path.** Not assumed from memory. Filing by primary subject, not format or skill name. (Mandate added see OPUS-4-7 tracker #27.)
- **Any file path cited INSIDE the doc content → verified to exist via `Filesystem:search_files`.** Inheriting a path from a skill ref, another doc, or memory ≠ verified. (Rule added after phantom-path cascade miss; see OPUS-4-7 tracker #39.)
- **Locating canonical home for an artifact (AP log, decision memo, audit log, doc home, AOS service spec, etc.) → semantic search query FIRST.** Directory walks (`list_directory`/`read_file` recon) are NOT a substitute. Familiar workstream ≠ excuse to skip search. (Rule added — enforcement; pattern first measured persisted many sessions without bite.)
- **Archiving a doc → verify it is actually done, not pending.** Format, age, or apparent completeness does NOT confirm a doc is archiveable. Pending-implementation docs (setup specs, migration guides, onboarding checklists) look "done" by format but are not. Require `status: COMPLETE` in frontmatter OR explicit Jordan confirmation before moving to `Archive/`.

FAIL ANY CHECK = STOP. Fix before proceeding.

### 1. GATHER KNOWLEDGE
- Search for existing docs on topic via semantic search
- If topic covered: UPDATE existing, don't create new
- Read `references/LAYER-TAXONOMY.md` for placement rules
- Read `references/AGENTIC-DOC-STANDARDS.md` for quality requirements

### 2. CREATE OR EDIT

**New doc:**
1. Determine layer (HAIOS/AOS/PB) per 3-Layer taxonomy
2. Apply template from `references/DOC-TEMPLATES.md`
3. Write content following agentic standards
4. If SSOT: add to `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md`

**Existing doc:**
1. Make changes
2. Bump version (minor: 1.0→1.1, structural: 1.x→2.0) — **but see "Version-bump discipline" under Content Rules: same-session iteration on the same change does NOT compound version bumps.**
3. Update date
4. If SSOT: check cascade rules in registry

**File move (single or batch):**
1. Execute the move(s).
2. **Path reference audit** — immediately after, search for every old path across in-scope docs AND script path constants:
   - Single move: `Filesystem:search_files` for the filename across `<drive-root>/`
   - Batch move (3+ files): list all old→new path mappings, search each in turn
   - **Script constants:** grep the moved filename(s) across `.py`/`.ps1` path constants in `HAIOS/Tools/` (and any other script dirs), not only `.md` cross-refs. (evidence: `CONFIG-SURFACE-INVENTORY.md` relocation updated doc refs but missed `eval_gate.py`'s hardcoded `INVENTORY` constant — next loop run would have STOPped; K0 fix in Phase 1.5 Stage 1.)
3. Fix every broken reference found before declaring the move complete.
4. Bump version + add changelog entry on any doc edited during the ref-fix pass.

*evidence: moving 15 files across AGENCY/ broke references in 8 docs; path audit discovered post-move and cost significant session time. Pre-commit audit surfaces scope upfront.*

**Skill edit** — route by skill track (see `references/LAYER-TAXONOMY.md` Skill Placement; doctrine split):

*COO skill* (canonical at `AOS/Skills/[skill]/`):
1. Edit the canonical file at `AOS/Skills/[skill]/SKILL.md` or `references/`.
2. Bump version if the skill tracks one in frontmatter.
3. Mandatory handoff: tell Jordan explicitly — "**Skill updated: `[skill-name]`.** Install on Claude Desktop from `AOS/Skills/[skill-name]/` when convenient. Change summary: [one line]."
4. `session-end-skill` will stage the entry under NEXT SESSION → "Skill installs pending" in session-context.md. Entry persists until Jordan confirms install.
5. DO NOT write to `HAIOS/Skills-Updates/` — that folder is deprecated as of then.

*CC-operational skill* (canonical at `~/.claude/skills/[skill]/` on Desktop, Drive mirror at `AOS/CC-Config/skills/[skill]/`):
1. Edit the canonical file at `~/.claude/skills/[skill]/SKILL.md` or `references/` via Filesystem tools. 8 skills in this track: `n8n-workflow-build`, `n8n-workflow-audit`, `n8n-diagnose`, `process-feedback`, `semantic-search`, `semantic-reindex`, `archive`, `github-update-skill`.
2. Do NOT edit the Drive mirror at `AOS/CC-Config/skills/` directly — `sync-cc-config.ps1` runs `/MIR` and overwrites out-of-band Drive edits at next sync.
3. Bump version if the skill tracks one.
4. **No "Skill installs pending" entry** — CC reads local, so it's live on save.
5. `session-end-skill` step 8.5 triggers the sync to mirror the edit to Drive (session-end cadence).
6. Mandatory handoff: tell Jordan — "**CC skill updated: `[skill-name]`.** Live immediately for CC; Drive mirror refreshes on next session-end sync. Change summary: [one line]."

### 3. VERIFY AGENTIC COMPLIANCE

Before completing, verify:
- [ ] Headers are concepts ("Database Schema" not "Section 1")
- [ ] All references use full paths (not "the schema doc")
- [ ] Paragraphs self-contained (chunk-friendly)
- [ ] Frontmatter complete (title, version, updated, purpose)
- [ ] If SSOT: registered with cascade rules
- [ ] Content matches doc purpose:
  - Rules/behavior: `claude-instructions.md`
  - Current state: `session-context.md`
  - How-to/process: skills
  - Architecture/design: design docs
  - Reference data: MASTER docs

Fail = fix before completing. Partial pass = note gaps in handoff.

## Handoff Contract

When complete, state:
- Files created/modified: [full paths]
- Status: success | partial | failure
- If SSOT changed: [cascade dependents identified]

## Dependencies

- Required: Desktop Commander, semantic search workflow (`{{WORKFLOW_ID}}`)
- Related: semantic-index-update-skill (index freshness after doc changes)

## Authority

Tier 2 (Inform after) — Internal doc changes, reversible

## Error Handling

- **Topic already covered:** Update existing doc, don't create duplicate
- **SSOT conflict:** Two docs claim same topic; stop, flag for Jordan
- **Cascade unclear:** List potential dependents, ask Jordan to confirm
- **Path uncertain:** Check 4-Layer taxonomy, ask if still unclear

## Content Rules (from that session Doc Audit)

These rules apply whenever COO writes or edits ANY doc in MRMINOR. They complement the PRE-FLIGHT checklist above and the VERIFY step.

### Frontmatter discipline
- Required fields on every doc: `title`, `version`, `updated`, `purpose` (or `description`)
- `updated` must reflect the actual session/date of this edit — not inherited from the source file
- Bump `version` on any substantive content change (minor: 1.0→1.1, structural: 1.x→2.0)

### Naming convention — self-locating filenames

A doc's filename must identify it **without its folder**. The flat `MRMINOR/Archive` (and cross-doc references, and search results) carry no path context — the name is the sole provenance carrier. A file you'd have to open the folder to understand is mis-named.

**Point-in-time artifacts** (diagnosis, decision, plan, audit, post-mortem, review, spec, framework — authored once, eventually archived):
`{SUBJECT}-{DOC-TYPE}-S{session}.md` — e.g. `ARCHIVE-ARCHITECTURE-DECISION.md`, `REMEDIATION-DUPLICATION-DIAGNOSIS.md`. Use `-{YYYY-MM-DD}` instead of `-S{session}` when a date is more meaningful.
- `{SUBJECT}`: specific enough to disambiguate (not `cleanup`, `update`, `notes`).
- `{DOC-TYPE}`: one of `diagnosis | decision | plan | audit | post-mortem | review | spec | memo | framework`.
- The session/date stamp makes the name unique in a flat store and disambiguates iterations.

**Living docs** (SSOTs, roadmaps, `session-context`, registries — continuously updated, never archived): stable canonical name, no session stamp (e.g. `DOC-GOVERNANCE.md`, `AGENCY-BUILD-ROADMAP.md`). Still self-locating (subject + type); version-tracked in frontmatter, not the filename.

**Banned:** generic names that lose meaning when flattened — `notes.md`, `plan.md`, `update.md`, `draft.md`, `temp.md`, `v2.md`. If the name only makes sense inside its current folder, rename before writing.

*Why: the archive collapsed to one flat `MRMINOR/Archive` (`HAIOS/Knowledge/ARCHIVE-ARCHITECTURE-DECISION.md`). Range subfolders and per-layer archives are gone, so the path no longer encodes which thing a file is. Retrieval is direct name lookup — only a self-locating name survives the move.*

### Version-bump discipline

Same-session iteration on a draft does NOT produce multiple version bumps in the changelog. Anti-pattern: a doc gets drafted, Jordan reviews mid-session, feedback triggers rebalancing, doc gets re-written — producing v1.3 → v1.4 → v1.5 in one session. Three changelog rows for one logical decision (the locked draft Jordan signs off on). The changelog should capture the locked state, not intermediate iterations.

**Rule:** Within a single session, in-session iteration before Jordan locks the version stays at the **target version** (the version the doc will become when locked). The changelog row at session-end captures the final locked state, not intermediate iterations. The frontmatter `updated:` line stays current session.

**Operational pattern:**
1. First write: assign target version (e.g., v1.5 for an upgrade from v1.4 baseline)
2. Mid-session feedback / revision: re-write to same target version, no intermediate bump, single changelog row evolves in place
3. Final write at session-end reflects locked state

**When to bump within a session (legitimate):**
- Jordan explicitly locks v1.5 mid-session and asks for v1.6 changes
- A separate, distinct change scope is opened (not iteration on the same change)
- Multi-session arc where each session lands a different version

**When NOT to bump (the discipline):**
- Re-writing the same section after Jordan feedback within the same drafting pass
- Rebalancing copy after a same-session review comment
- Aligning cross-refs after a related doc was updated in the same session

*evidence: UPWORK-FIVERR-POSITIONING.md went v1.3 → v1.4 → v1.5 in one session because Jordan reviewed v1.4 mid-draft and requested rebalancing for both buyer paths. Same with B1-WORKFLOW-AUTOMATION-BUILD.md v1.0 → v1.1 → v1.2 (initial draft → rebalance → ladder-ref alignment). Three changelog rows each. With this discipline, both would have produced single changelog rows reflecting the actually-locked v1.5 / v1.2 state.*

### Authority hierarchy — check before writing any content

Before writing, ask: **"does this belong higher?"** Canonical rubric — the Best Practice → Strategy → Architecture → Design hierarchy plus per-layer examples — lives at **`HAIOS/Knowledge/DOC-GOVERNANCE.md §Authority Hierarchy`** (SSOT). Do not duplicate inline. Operational application for writes:

- Content applies cross-workflow → do NOT write in workflow README; find canonical location or flag for COO
- Content is strategic → write in workstream master or ROADMAP.md, not in operational docs
- Content is architectural → write in HAIOS/Architecture/ or AOS/Architecture/
- Content is a best practice → write in HAIOS/Knowledge/BestPractices/ or CC rules

### No duplication
- Before adding any fact: check if it's stated elsewhere. If yes → link, don't duplicate.
- Before creating a new doc: verify no existing doc covers this content (semantic search first).

### Workflow README specifics
When creating or editing a workflow README:
- `status` field required: LIVE / INACTIVE / ARCHIVED / PLANNED / DEPRECATED
- `version` in frontmatter must match n8n workflow name suffix (e.g., V3 in n8n = `version: V3` in frontmatter). Fix the mismatch while in the file.
- Status must match live n8n `active` field — check before writing
- Future/Planned section: only include items NOT already tracked in any domain roadmap. If the item is in a roadmap → strip it, no pointer needed.
- No build queues, no architectural principles that apply beyond this workflow

### Strategy doc discipline
- Verify content is a current decided position, not an open question
- Open questions go in domain roadmaps as build items, not in strategy docs as TBDs

### Cross-reference verification
- Verify every file path written actually exists (`Filesystem:search_files` — not assumed from memory)
- Never write `/mnt/skills/user/` paths in Drive docs — those are container paths, not Drive paths
- Correct path prefix for skill files in Drive docs: `AOS/Skills/`

### Derive, don't assert — the depth rule

**Every value written into a doc is computed from the artifact, never recalled.** Version numbers, changelog placement, file counts, row counts, dates, path targets. If it can be read, read it. If it can be counted, count it with a tool.

**Context depth does not change the rule — it changes the stakes.** Deep in a session, recall degrades while confidence does not. That gap is the failure: a remembered version number *feels* exactly as certain at 200k tokens as at 20k, and is far likelier to be wrong.

> **Depth doesn't stop the work. Depth withdraws the license to trust yourself.**

**The check, per value:** *did I compute this, or recall it?* Recall → go read it. One grep is cheaper than one wrong changelog row, and enormously cheaper than a wrong row that ships.

| Value about to be written | Derive it from |
|---|---|
| Version number | `grep` the file's own header / frontmatter |
| Changelog placement | `grep` the adjacent rows — confirm the new row lands where the ordering says |
| Files / rows touched | the enumerated source (the row table, `unzip -l`, `wc`) — never a count by eye |
| A path cited in content | `Filesystem:search_files` (Rule 9) |
| "the file now says X" | read it back after the write |

**Do NOT queue the work instead.** Deferring mechanical work to a cascade *on depth grounds* re-creates the leak `MODEL-ROUTING-PROTOCOL` §Batch-size threshold closed at that time: late-session incidental repairs are inline work by default — the batch is small and the discovery does not survive a handoff. The fix is verification, not deferral.

**Evidence:** three consecutive version errors on one changelog late in a session (wrong number ×2, wrong placement), plus two file-count misestimates. Every one was a value asserted from memory rather than read from the artifact. Depth raised the rate; asserting-instead-of-deriving was the mechanism. **Counter-evidence:** the same work class — 11 surgical edits across 2 Drive files, 2 version bumps, 2 changelog placements, a file count — at comparable depth, zero errors, because every value was grepped, `dryRun`-ed, or hashed before it was written.

**Same mechanism, already live elsewhere:** `cc-prompt-skill` §2 (derive counts from the enumerated source; verify exact-text assertions via Grep) and its `references/PROMPT-TEMPLATE.md` §COO PRE-WRITE GATE (node count = `len(nodes)`, never eyeball-scan — v1.26). This section is the doc-authoring instance of one rule that had been re-derived per artifact type, and re-failed at each new one.

### Fix-on-sight (Jordan standing instruction)

**A stale reference found is a stale reference fixed — in the session it surfaces, not queued.** Finding a broken path and reporting it without repairing it leaves the next reader to rediscover it, and the discovery cost is paid again every time.

**Applies to:** broken/moved paths, renamed folders, retired conventions still cited as live, doc references to deleted assets, version claims contradicting the file's own frontmatter.

**Procedure:**
1. **Verify before fixing.** Confirm the true target via `Filesystem:search_files` — never fix a path from memory, or you replace one wrong path with another.
2. **Fix the whole family, not the one hit.** A path that moved once is usually cited in several places in the same file. Grep the file for the old base before declaring it fixed.
3. **Bump version + changelog row** naming the trigger and listing every corrected reference.
4. **Skill edits** → mandatory install handoff (see §Skill edit).

**Do NOT fix on sight when:** the reference is *contested* rather than stale — i.e. two live docs disagree about a canonical home, or the correct target is a Tier 1 judgment (SSOT reshuffle, canonical rehoming). Those are audit findings for Jordan, not repairs. Silently picking a side is a doctrine decision made by accident. *(Note: `SKILLS-ROADMAP.md` is claimed by both `AOS/Skills/` and `HAIOS/Architecture/` — left as a live Q3-2026 Tier 1 decision, while 10 genuinely-stale paths in the same skill were repaired.)*

*evidence: `quarterly-doc-audit-skill` v2.4 pointed every artifact path at `HAIOS/Architecture/DOC-AUDIT-Q{N}-{YEAR}/`. The artifacts had moved to `AOS/Workflows/HAIOS - Infra - Doc Governance [L2]/`; zero remained at the cited base. The skill that exists to detect drift had been carrying its own broken paths since then — the drift outlived the audit designed to catch it.*

### Correct but pure cost — check lifecycle before firing an obligation (Jordan; n=6 at that time)

**A mechanical obligation is owed to an artifact that will still exist to be read.** §Fix-on-sight says repair now rather than queue. This is its counterweight: some obligations should not fire at all.

Rules like *queue the semantic-index update on any `MRMINOR/` write*, *bump the version*, *add the changelog row*, *register the SSOT* are keyed on **path**. A path predicate says nothing about whether the artifact outlives the obligation. Fire one on an ephemeral file and the work is **correct by the letter and pure cost in fact.**

**The predicate:** *will this artifact still be read after today?*
- **Durable** (living doc, spec, strategy, skill, registry) → the obligation stands.
- **Ephemeral** (a CC prompt archiving within hours, a scratch cascade, a decision checkpoint, any artifact whose own procedure ends in `Archive/`) → skip it, **and name the skip.** A skipped obligation that is stated is a decision; a silently skipped one is drift.

**This is not permission to skip work that feels tedious.** The predicate is lifecycle, not effort. A tedious obligation on a durable doc still fires.

| n | Obligation that fired | Why it was pure cost |
|---|---|---|
| 5 | Rule 2 queued a semantic-reindex CC prompt | The indexed file archives within hours — the index entry dies before it is ever queried. Rule 2 is a **path** predicate and needs a **lifecycle** one. |
| 6 | Post-install checklist required a `SKILLS-ROADMAP` version backfill | The version column has **no consumer** (verified: every reference is a writer, or reads the table for existence/backlog). Backfilling 15 versions across 5 skills yields a momentarily-accurate field nothing reads, which drifts again at the next install. Docked to the Q4 slate, where the same doc's home conflict is already pending — see §Fix-on-sight, *contested ≠ stale*. |

**The liability test (Jordan — retired the `ANTI-PATTERNS.md` active-entry count):** a field with **(a)** no consumer, **(b)** a per-addition edit cost, and **(c)** a demonstrated drift record is a liability, not a record. Three for three → raise it for a ruling, don't refresh it. *"What was the number for?"* is the entire audit.

### Sister-artifact reconciliation

When authoring 2+ related docs in the same session that share a schema, taxonomy, contract, enum, or any other contract surface (specs that write to the same table; a strategy doc + its implementation spec; A/B/C-style paired artifacts): do an in-session reconciliation pass before declaring any of them complete.

**Pattern:**
1. Draft each artifact to v0.1 with whatever assumptions are clearest at the time — don't pre-block on cross-artifact alignment.
2. After the last artifact in the set lands, run a reconciliation pass across the set.
3. Identify mismatches: same field named differently, drifted enum values, contradictory contract semantics, divergent state-model assumptions.
4. Fix in the sister(s) AND in the originating doc; log the reconciliation in each changelog row (`Reconciliation with sister artifact X.md applied same-session: (a) ..., (b) ...`).
5. After reconciliation, treat the set as a single locked release — same version number across sisters where the contract maps 1:1, or distinct versions with explicit cross-version-pin notes.

**Why in-session, not cross-session:**
- In-session memory of source-of-truth is sharp; cross-session reconciliation costs roughly 2x (re-loading context, re-searching for canonical values).
- Drift in v0.1 propagates to downstream consumers (CC build prompts, cascade tasks, semantic-search results) before next session catches it.
- Single changelog row per artifact captures the reconciliation cleanly; deferred fix produces a separate "v0.1 → v0.2 — sister-alignment fix" row that's lower-signal noise in the version history.

**Cost:** ~3–4 edits per artifact + a verification read; trivial vs the cross-session alternative.

*evidence: INTAKE-FORM-SPEC.md (B) drafted before AGENCY-PIPELINE-SCHEMA-SPEC.md (C). B seeded with non-canonical source-enum values (raw labels not the LOCKED 9-channel taxonomy) + an incorrect Stage-0 state model. C drafting surfaced canonical source taxonomy + "no Stage 0 in pipeline" doctrine. Same-session B↔C reconciliation: 4 targeted edits in B against 2 changes (source-enum mapping, stage-model correction), single changelog row appended. Sister artifact A had already landed; A↔B reconciliation in the prior session pattern followed the same shape. Pattern emerged across + as a reliable multi-artifact authoring approach.*

#### OPEN-N resolution timing

When an OPEN-N item surfaces during reconciliation:
- **Resolve same-session** when: (a) current session is Opus, and (b) the item is decidable from context already loaded.
- **Defer** when: (a) resolution requires fresh state not in current context, or (b) resolution requires stakeholder input not yet provided.
- **Do not auto-defer** to "next Opus session" when both same-session conditions are met — that adds latency with no benefit.

*Rationale: Opus-now and Opus-later are the same agent on the same context. When the judgment is available, use it. Defer is a cost, not a default. (Observed codified.)*

### Session artifact routing
- Never land `CC-PROMPT-*`, `CC-BUILD-LOG-*`, `CC-FEEDBACK-*` in canonical dirs.
- Archiving (CC artifacts and retired COO docs alike) = move to the ONE flat `MRMINOR/Archive` by path, with a self-locating name (§Naming convention). Never to range subfolders, never to a per-layer `Archive/`. CC follows the `archive` skill (`~/.claude/skills/archive/`); COO follows the same contract. (Replaces the old `Archive/CC-Prompts-` convention — see `HAIOS/Knowledge/ARCHIVE-ARCHITECTURE-DECISION.md`.)

### Doc Lifecycle — COO-Readable Standard

Docs in MRMINOR exist in one of two states. The transition between them is a structured event, not an aspiration.

**State 1 — Jordan-review (`review-state: pending-jordan` in frontmatter).**
Written human-readable for Jordan's evaluation. Narrative prose, rationale paragraphs, full historical context, validation stories, why-we-decided-this. This is the natural state of any doc during a strategy arc or new sub-strategy build-out — Jordan needs to evaluate the *reasoning*, not just the *conclusion*.

**State 2 — COO-operational (`review-state: ratified` in frontmatter; or default for COO-exec docs that never had a Jordan-review state).**
Decisions + pointers + structured data. No rationale paragraphs in live sections. ≤2-hop access to any answer. Schema: enums, LOCKED/OPEN/DEFERRED, tables, bullets, pointer-to-SSOT for any concept that has a canonical home elsewhere. Changelog: last 3 inline; older → sidecar `CHANGELOG-[DOCNAME].md`.

**Transition trigger: Jordan approval event.**
When Jordan ratifies a doc (verbal approval in chat, explicit "locked" / "approved" / "ratified"), the doc enters compression eligibility. The compression pass is **not optional** — approved docs become operational docs. Leaving an approved doc in Jordan-review prose state creates the bloat that necessitated streamlining.

**Execution: Sonnet cascade.**
Compression pass = Opus reads + specs (action table per section: KEEP / COMPRESS / STRIP / POINTER / ARCHIVE) → Sonnet executes the mechanical surgery. Opus owns judgment; Sonnet owns the find/replace. Cascade file lives at `HAIOS/Sonnet-Cascade/S{N}-{topic}-cascade-completion.md`. see the agency-docs streamlining cascade as the reference pattern: `HAIOS/Sonnet-Cascade-agency-docs-streamlining-cascade-completion.md`.

**Compression pass strips:**
- Rationale prose paragraphs explaining *why* a decision was made → belongs in changelog
- Historical narrative (what happened in session) → changelog or archive
- Content that duplicates a sub-strategy SSOT → replace with pointer
- Validation stories ([client]-specific learnings) → archive or BUYER-PSYCHOLOGY (where they belong as doctrine)
- Lengthy §Methodology / §Background prose explaining *purpose of the doc* → archive if not a live operational rule
- Changelog entries beyond last 3 → sidecar `CHANGELOG-[DOCNAME].md`
- Per-SKU / per-channel / per-section "why this exists" paragraphs → strip (rationale belongs in changelog)

**Compression pass keeps:**
- Frontmatter (complete: version, status, SSOT-for, upstream, downstream, gates)
- Enumerated decisions with 1-line rationale max (LOCKED/OPEN/DEFERRED)
- Tables + bullet lists of live operational content (entry/exit criteria, pricing ladders, lane structures, checklists)
- Pointers to sub-strategy SSOTs
- Last 3 changelog entries
- Content unique to this doc that can't be found by following a pointer

**Build items doctrine.**
Anything identified as a to-do during doc authoring or Jordan review gets added to `AGENCY/Knowledge/AGENCY-BUILD-ROADMAP.md` (or domain equivalent) **immediately** — not left in the strategy doc as a checkbox list. Strategy docs describe current decided state; roadmaps describe future work. Mixing them produces the duplication problem streamlining fixes.

**When NOT to compress:**
- Doc is still actively iterating (Jordan hasn't ratified the current version)
- Doc is a working scratchpad, not a strategy or operational SSOT
- Doc's purpose is *itself* historical (audit reports, post-mortems, archived references)

*evidence: AGENCY strategy arc produced ~480-line AGENCY-STRATEGY, ~750-line AGENCY-BUILD-ROADMAP, ~900-line MARKETING-STRATEGY, ~380-line OFFERINGS-STRATEGY — all written for Jordan-review, all ratified, all left in pending-jordan state. Aggregate ~2,500+ lines of strategy content where ~1,400 lines was operational. streamlining cascade compresses to ~1,500 lines (~40% reduction) by stripping rationale prose + historical narrative + sub-strategy duplicates. Doctrine codified here so the pattern doesn't recur on the next arc.*

### Ratification audit

Before toggling `review-state: pending-jordan` → `ratified` on any non-roadmap doc: grep the doc for to-do patterns. Disposition each hit — never ratify with buried items in prose.

**Patterns to detect:**
- `TODO`, `TBD`, `TBC`, `XXX`, `FIXME`
- `next step`, `eventually`, `someday`, `down the road`
- `pending`, `we should`, `we need to`, `we'll need to`, `we'd like to`
- Unchecked checkboxes `- [ ]`
- Operational status emojis: `🔁`, `⏸️`, `🟡`, `🔴` (excludes `✅` `🟢` `🟠` which describe state, not pending action)
- `OPEN-N:` blocks (see §OPEN-N closure rule below)

**Disposition for each hit:**

| Type | Action |
|---|---|
| Build/delivery/operational to-do | Migrate to AGENCY-BUILD-ROADMAP (or domain equivalent) → replace inline with pointer to roadmap row |
| Decision question | Track as `OPEN-N:` with closure rule applied at ratification |
| Stale / irrelevant | Strip from body |
| Resolved in ratification convo | Capture decision in changelog row, strip from body |

**Changelog must document:** `Ratification audit: N migrated / N tracked-OPEN-N / N resolved / N stripped` (zeros omitted, e.g. "Ratification audit: 2 migrated, 1 resolved").

**Exception:** Roadmap docs themselves (`*-BUILD-ROADMAP.md`) — to-do patterns ARE the live content, no audit.

*evidence: B4-AI-INTEGRATION v1.0 ratification (Jordan approved) had no pre-ratification scan; ratified without audit. No buried items found post-hoc, but the absence of the audit was the pattern Jordan flagged. Doctrine added so future ratifications don't depend on absence-of-bugs.*

### OPEN-N closure rule

Every `OPEN-N:` block in a `review-state: ratified` doc carries exactly one state:

- **RESOLVED** + 1-line decision + session ref (e.g., `OPEN-3 RESOLVED — chose Option B because [reason]`)
- **DEFERRED** + roadmap link (e.g., `OPEN-3 DEFERRED — see AGENCY-BUILD-ROADMAP P2-OFF-04`)
- ACTIVE is **NOT a valid state** in a ratified doc. ACTIVE-in-ratified = doctrine violation.

If ratification audit surfaces ACTIVE `OPEN-N:`: resolve in ratification convo or defer to roadmap before flipping `review-state: ratified`.

### Completion sync

Quick Rule 3 (claude-instructions): tracker → DONE only after CC completes + COO verifies + Jordan reviews. This rule adds: **the roadmap update is atomic with verification**, not a deferred operation.

**Atomic sequence:**
1. CC reports completion
2. COO verifies (re-run / output check / endpoint test)
3. **Same operation:** edit relevant roadmap row → `status: DONE` + completion session ref + verification note
4. Then surface to Jordan for final review

**Not acceptable:**
- "I'll mark it DONE next session"
- "Let me batch the roadmap updates at session-end"
- Marking DONE before verification (rubber-stamping)

**Backstop:** session-end-skill drift audit §Completion-sync drift (#50) flags any verified-but-not-marked-DONE items.

For cascade-pre-staged work (Opus specs → Sonnet executes), see §Cascade-pre-staged completion sync below.

### Cascade-pre-staged completion sync

§Completion sync atomicity applies at the CC-execution boundary. It does NOT cover **cascade-pre-staged work**: mechanical surgery (find/replace, version bumps, path-refs, doc annotations) that Opus pre-stages in `HAIOS/Sonnet-Cascade/S{N}-{topic}-cascade-completion.md` for next-session Sonnet execution (claude-instructions Quick Rule 12).

The atomicity principle still holds — just shifted to the Sonnet boundary.

**Atomic sequence (cascade variant):**
1. Opus authors cascade file with per-item instructions + **explicit roadmap-update items** for any work tracked in a roadmap (e.g., "Edit `AGENCY-BUILD-ROADMAP.md` row P1-MKT-04 → `status: DONE` + S{N+1} ref")
2. Sonnet executes cascade items in sequence
3. **Same operation:** Sonnet flips each roadmap row → `status: DONE` + cascade-execution session ref + cascade-completion path as verification trail
4. Sonnet appends cascade-completion → `status: COMPLETE` (or `PARTIAL` if items failed/skipped or `## Cascade-Discovered Items` appendix non-empty per §Cascade discovery protocol)
5. Opus reviews cascade-completion at next session = "Jordan reviews" backstop step

**Opus authoring discipline:**
- Every roadmap-tracked item in cascade scope MUST appear as an explicit cascade item, INCLUDING its roadmap edit
- DO NOT pre-mark roadmap DONE at cascade-authoring time — that's rubber-stamping (work not executed)
- Roadmap row stays at current status (IN PROGRESS / PLANNED) until Sonnet flips it atomically

**Sonnet execution discipline:**
- Cascade items + roadmap updates execute in the same pass — never defer "Opus will mark it next session"
- Partial completion: failed/skipped items leave corresponding roadmap rows at prior status; cascade-completion lists per-item disposition (DONE / SKIPPED / FAILED-reason); status `PARTIAL`

**Not acceptable:**
- Opus pre-marking roadmap DONE before cascade executes
- Sonnet completing cascade work without atomically applying roadmap-update items
- Roadmap row remaining IN PROGRESS after cascade-completion `status: COMPLETE` for that item

**Backstop:** session-end-skill drift audit §Completion-sync drift (#50) extends to: cascade-completion files with `status: COMPLETE` whose constituent roadmap-update items did not propagate = flagged.

*evidence (pre-emptive codification): §Completion sync v2.10 written for CC-execution actor boundary. Cascade-pre-staged model (Opus specs → Sonnet executes) operates on the same atomicity principle at a different boundary. Two failure modes without explicit clause: (a) Opus pre-marks roadmap DONE at cascade-authoring time (rubber-stamping), (b) Sonnet executes cascade but defers roadmap update. Both reproduce the deferred-update failure §Completion sync exists to prevent. No production violation observed yet.*

### Cascade discovery protocol

Trigger: any Sonnet compression/streamlining cascade.

While compressing each doc, scan for the same patterns from §Ratification audit detection list. If found AND being stripped or compressed away in the pass: flag into a `## Cascade-Discovered Items` appendix at the bottom of the cascade completion file (above §Cascade Completion).

**Each flagged item:**
- Source path + section
- Verbatim pattern (so Opus can disposition without re-grepping)
- Suggested disposition: `migrate` / `track` / `resolve` / `strip`
- Sonnet rationale (1-line) for the suggestion

**Cascade exit state:**
- Appendix empty at cascade-end → `status: COMPLETE`
- Appendix has items → `status: PARTIAL — pending Opus appendix disposition`. Opus reviews next session, dispositions each item, then closes cascade.

**Why:** Compression removes context. Buried to-dos in compressed prose vanish silently. The appendix is the safety net for the §Doc Lifecycle compression pass.

**Self-resolved judgment calls count as findings, not clean passes.** If a dispatched audit subagent (or Sonnet itself) finds a borderline compliance hit and constructs its own reasoning for why it's "not a violation" — reasoning that isn't stated anywhere in the criteria text it was given — that reasoning-to-explain-away IS the judgment call. Log it to the appendix for human/COO disposition; do not resolve it in the audit report as a clean pass. When writing an audit/inspection dispatch prompt (Phase-2-style, checklist-driven, subagent-executed), add an explicit line: "If you find yourself explaining via reasoning not stated in the criteria text why a hit is NOT a violation, treat that as a judgment finding, not a clean pass — log it, don't resolve it." *Evidence: (agency-realignment Phase 2 cascade) — 2 of 7 parallel audit subagents independently rationalized a content-doctrine tension ("AI never writes content prose" vs. an AI-assisted-drafting-with-approval-gate workflow) as compatible with doctrine rather than escalating it, despite an explicit "judgment finding: don't edit, log it, do not guess" instruction; caught only by a post-run backstop grep, not by the subagents' own reports (see W65, `ANTI-PATTERNS.md`).*

## Tool Quirks (COO layer)

Known failure modes in Filesystem + Desktop Commander tools. Read before any multi-section file edit.

### MCP G:-drive access loss mid-session

**Symptom:** Filesystem tools that were reading/writing the shared drive paths suddenly return "Access denied - path outside allowed directories" listing only the shared drive dirs; companion write tools may return "tool not found."

**Cause:** the G:-scoped MCP server disconnected mid-session while a second, C:-scoped Filesystem server stayed connected and answers under similar tool names — the failure masquerades as a permissions change.

**Diagnostic:** call Filesystem:list_allowed_directories. If the shared drive is absent, the G:-capable server is down (config did not change; the server did).

**Recovery:** Jordan restarts Claude Desktop / reconnects the connector. Verify the shared drive reappears in list_allowed_directories BEFORE resuming Drive writes.

**Interim protocol:** stage unavoidable in-flight work to an allowed the shared drive dir WITH a relocation banner at top; on recovery, write canonical to the shared drive fresh (not a move — refresh frontmatter/status), then Jordan deletes the staging copy. Never treat the shared drive staging as canonical; never file new work to the shared drive silently.

### Filesystem:edit_file -- backtick-dollar injection

**Bug:** If newText contains a backtick immediately followed by a dollar sign (e.g., markdown inline code referencing n8n variables like $json, $input, $workflow), Filesystem:edit_file injects the entire file content at that position, corrupting the file. The edit appears to succeed (no error thrown) but the file grows by the full file size worth of duplicate content.

**Trigger pattern:** Any newText string containing the two-character sequence: backtick followed by dollar sign.

**Workaround (in priority order):**
1. Use Filesystem:write_file with the complete corrected file content (safest for large sections)
2. Use Desktop Commander:edit_block with old_string/new_string (handles special chars better)
3. If Filesystem:edit_file must be used: split the code span across the edit boundary, or substitute a placeholder and fix in a second pass without triggering the pattern

**Detection:** After any edit where newText contained backtick+dollar sequences, verify file size did not double. Quick check: read tail=5 and confirm the file ends normally.

**Recovery:** Filesystem:copy_file_user_to_claude -> Python to find and remove the injected block -> Filesystem:write_file clean content back. see the session notes n8n-workflows-core.md repair as reference procedure.

**Note when documenting this bug:** Do NOT write backtick+dollar literally in any newText when editing this section -- use "backtick followed by dollar" or "backtick-dollar" in prose instead.

### Filesystem:search_files — glob scope

**Bug:** Bare glob patterns like `*.md` in `Filesystem:search_files` do NOT recurse into subdirectories — the pattern matches only at the directory level named by the `path` parameter.

**Workaround:** Use `**/*.md` (double-star) for recursive search across subdirectories. Caveat: confirm whether match paths returned are relative to the `path` root or absolute, as behavior can vary.

**Failure mode:** A search for `*.md` in `<drive-root>/` returns only top-level `.md` files, silently under-reporting matches in subdirs. Path reference audits and doc searches are especially vulnerable — a false "not found" leads to declaring a reference clean when broken refs exist deeper.

### Filesystem:edit_file — non-ASCII glyph anchor mismatch

**Pattern:** when oldText contains an uncommon Unicode glyph (flag / star / symbol markers), reproducing the glyph from memory silently fails to match — visually similar glyphs (black flag vs lightning vs helmet) are distinct codepoints, and the error says only "no exact match."

**Rule:** before any edit whose oldText contains a non-ASCII glyph, read the exact line first (read_text_file head/tail) and copy the anchor verbatim. Never type the glyph from memory.

**Atomicity note (confirmed):** a multi-edit batch with one failed anchor applies ZERO edits — re-read file state before retrying, but no partial-application cleanup is needed.

**Evidence:** — two consecutive edit batches failed on a flag-marker anchor (two different wrong codepoints); third attempt with a verbatim-copied anchor succeeded; frontmatter re-read confirmed the failed batches applied nothing.

## Cross-References

- Standards source: `HAIOS/Knowledge/BestPractices/KNOWLEDGE-ORGANIZATION-BEST-PRACTICES.md`
- Doc governance + detection patterns: `HAIOS/Knowledge/DOC-GOVERNANCE.md`
- Detail in: `references/AGENTIC-DOC-STANDARDS.md`, `references/DOC-TEMPLATES.md`, `references/LAYER-TAXONOMY.md`

## Changelog

| Version | Changes |
|---|---|
| 2.23 | **Trigger rewritten — this skill's `description` was the mechanism that produced unrequested files.** The old trigger fired on output *shape*: "producing an analysis, plan, diagnosis, framework, review, assessment, audit, decision memo, or any structured multi-section response." Because it lives in `description`, it auto-fires — so any structured answer routed here, and `claude-instructions.md`'s matching ENFORCEMENT block defaulted to **YES, write to Drive**. Together they did not merely permit unrequested docs, they **required** them, and nothing anywhere in the system said not to create a file Jordan had not asked for. Jordan asked COO to *display* an open-items review in chat; COO wrote `HAIOS/OPEN-ITEMS-REVIEW.md`. He deleted it and said he was afraid to look in the filesystem. New trigger keys on the **request** — a doc Jordan asked for, a named output path, an ongoing artifact he will reopen — and states explicitly that a review or analysis he asked to SEE is a chat answer. Lockstep with `claude-instructions.md` v25.40 §ENFORCEMENT and `session-end-skill` v1.21 drift check #49 (now bidirectional). |
| 2.22 | **Two Content Rules added, both Jordan-directed, both about obligations that fire correctly and cost more than they return.** (1) **§Derive, don't assert — the depth rule.** Every value written into a doc is computed from the artifact, never recalled: version numbers, changelog placement, counts, dates, path targets. Context depth does not change the rule, it changes the stakes — recall degrades at depth while confidence does not, and that gap is the failure. Explicitly rejects the candidate framing (*"Opus stops surgical edits past a depth threshold and queues them"*) on two grounds: it would re-create the leak `MODEL-ROUTING-PROTOCOL` §Batch-size threshold closed at that time (late-session incidental repairs are inline work by default — Jordan overruled the same instinct twice), and it treats the rate rather than the mechanism. Evidence: threw 3 consecutive version errors on one changelog (wrong number ×2, wrong placement) + 2 file-count misestimates, every one a value asserted from memory. Counter-evidence: ran the same work class at comparable depth — 11 surgical edits / 2 files / 2 version bumps / 2 changelog placements — with zero errors, because every value was grepped, dry-run, or hashed first. Generalizes gates already live in `cc-prompt-skill` §2 (exact-text grep derive-counts) and `PROMPT-TEMPLATE.md` §COO PRE-WRITE GATE (`len(nodes)`, v1.26) — one rule that had been re-derived per artifact type and re-failed at each new one. (2) **§Correct but pure cost — check lifecycle before firing an obligation** (n=6). The counterweight to §Fix-on-sight: path-keyed obligations (reindex queue, version bump, changelog row, SSOT register) say nothing about whether the artifact outlives the obligation. Predicate = *will this still be read after today?* Durable → fires. Ephemeral → skip, **and name the skip**. Not a tedium exemption; the predicate is lifecycle, not effort. n=5: Rule 2 queued a semantic-reindex CC prompt for a file that archives within hours — Rule 2 needs a lifecycle predicate, not just a path one. n=6: the post-install checklist demanded a `SKILLS-ROADMAP` version backfill; the column was verified to have **no consumer** (every reference is a writer or reads for existence/backlog), so 15 backfills across 5 skills would produce a momentarily-accurate field nothing reads. Docked to the Q4 slate alongside that doc's already-pending home conflict. Carries forward Jordan's liability test (no consumer + per-addition cost + drift record = raise it, don't refresh it). |
| 2.21 | **Added §Tool Quirks: Filesystem:edit_file non-ASCII glyph anchor mismatch.** oldText anchors containing uncommon Unicode glyphs must be verbatim-copied from a fresh read, never reproduced from memory — visually similar glyphs are distinct codepoints and fail silently as "no exact match." Also records batch atomicity: one failed anchor in a multi-edit call applies zero edits. Evidence: two failed batches on a flag-marker anchor before verbatim copy succeeded. |
| 2.20 | **Added §Cascade discovery protocol addendum — self-resolved judgment calls count as findings, not clean passes.** Dispatched audit subagents that construct their own reasoning for why a hit is "not a violation" (reasoning absent from the criteria text they were given) must log it as a judgment finding, not resolve it as clean. Applies to all future Phase-2-style checklist-driven audit dispatch prompts. Evidence: Phase 2 cascade — 2 of 7 subagents self-resolved a content-doctrine tension rather than escalating (see W65, `ANTI-PATTERNS.md`). Drafted by CC (Tier 1, feedback-processed report), applied by COO after review. |
| 2.19 | Removed 1 stale a former e-commerce project ref (decommission cascade). |
| 2.18 | **Added §Fix-on-sight to Content Rules** — Jordan standing instruction: stale references get repaired in the session they surface, not reported and queued. Codifies verify-before-fixing (search, don't fix from memory), fix-the-whole-family (grep the file for the old base — a moved path is usually cited several times), version+changelog discipline, and an explicit **contested ≠ stale** carve-out: where two live docs disagree on a canonical home, that's a Tier 1 audit finding for Jordan, not a repair — silently picking a side is a doctrine decision made by accident. Evidence: `quarterly-doc-audit-skill` v2.4→v2.5 same session (10 stale paths repaired; `SKILLS-ROADMAP.md` home conflict deliberately left for the Q3-2026 Tier 1 slate). |
| 2.17 | **Added §Tool Quirks: MCP G:-drive access loss mid-session.** G:-scoped server can disconnect mid-session while a C:-scoped Filesystem server remains, masquerading as a permissions change. Diagnostic = list_allowed_directories; recovery = Jordan restarts Claude Desktop + verify the shared drive present before Drive writes; interim = banner-marked the shared drive staging, canonical rewrite on recovery, Jordan deletes staging. Evidence: outage during cascade-spec authoring. |
| 2.16 | Skill-count drift fix (fix-item 4): CC-operational track 6 → 8 (added `archive` + `github-update-skill`). Surfaced by tier_misplace detector. |
| 2.15 | **Extended §File Move ref-audit to script path constants (T3) + added §Tool Quirks: Filesystem:search_files glob caveat (T4).** (1) File Move path ref-audit step 2 now covers `.py`/`.ps1` hardcoded path constants in `HAIOS/Tools/` (and any other script dirs), not only `.md` cross-refs. Evidence: `CONFIG-SURFACE-INVENTORY.md` relocation updated doc refs but missed `eval_gate.py` `INVENTORY` constant (Phase 1.5 K0 fix). (2) New Tool Quirks subsection: bare `*.md` glob in `Filesystem:search_files` matches only named dir level, not subdirs — use `**/*.md` for recursive; confirm path relativity. Sonnet cascade Tasks 3+4. |
| 2.14 | **Added §Naming convention — self-locating filenames** + updated §Session artifact routing to the flat-archive model. Doc filenames must identify the doc without its folder (point-in-time artifacts: `{SUBJECT}-{DOC-TYPE}-S{session}.md`; living docs: stable canonical name). Routing now points at the one flat `MRMINOR/Archive` + the `archive` skill, replacing `Archive/CC-Prompts-`. Rationale: archive collapsed to one flat cold store (`HAIOS/Knowledge/ARCHIVE-ARCHITECTURE-DECISION.md`); path no longer carries provenance, so the name must. |
