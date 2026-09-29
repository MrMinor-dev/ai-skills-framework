---
name: session-end-skill
version: 1.23
updated: 2026-07-27
description: "Close session cleanly. Triggers: 'end session', 'wrap up', 'save progress', approaching context limit."
---

# Session End

Close session cleanly; save context, report changed files, verify cascade updates.

## Workflow

### 1. COLLECT SESSION SUMMARY
Gather from current session:
- Session number (from session-context.md)
- Focus area worked on
- Key outputs (files created/modified, decisions made)
- Next session tasks (if discussed)

### 2. CROSS-STREAM DEPENDENCY CHECK
Before writing session-context.md, ask internally:
- Did work in **[focus area]** create a dependency, blocker, or impact on another workstream?
- Examples: Job app work revealed a skill gap to address in HAIOS. Adapting deliverable reuses AOS tooling. Agency content research changed strategic priority ordering.

**If yes →** note the impact in the affected workstream's section when updating session-context.md. Format: `[Cross-stream]: [impact]`
**If no →** proceed.

**Roadmap dependency gate:** If any domain roadmap was edited this session, verify:
1. Does the new/changed item create a dependency on another domain?
2. Does it invalidate or alter an existing cross-domain dependency?
3. If yes to either → update the Cross-Domain Dependency Map in the master index (`HAIOS/Architecture/BUILD-ROADMAP.md`).

### 3. DEDUPLICATION + SIZE GATE
Before writing session-context.md, enforce structure and size:

**Deduplication test (do this FIRST):**
1. For each item you're about to write, check: does it appear in 2+ sections?
2. If yes → keep in ONE canonical home, remove from others:
   - Dated deadlines/meetings → FLAGS only
   - Next actions → NEXT SESSION only
   - Current status → WORKSTREAMS only
   - **Session narrative → RECENT SESSIONS only.** The header `**Last Updated:**` field is a POINTER (`S{N} — see RECENT SESSIONS table below for detail`), never a restatement. The full narrative lives in the RECENT SESSIONS row; duplicating it into Last Updated as prose is the same failure the dedup test exists to catch, and it compounds every session since the row never gets pruned out of Last Updated the way old RECENT SESSIONS rows age out. (Evidence: Last Updated had accumulated a full paragraph restating the RECENT SESSIONS rows verbatim — caught only because Jordan asked "anything duplicated?" after close.)
3. Section purposes (also documented in session-context.md HTML comment):
   - **NEXT SESSION:** Actionable items only. Priority queue + Jordan actions. No status, no context.
   - **WORKSTREAMS:** Current state. 1-line per sub-item. Status-oriented, not action-oriented.
   - **FLAGS:** Persistent time-sensitive alerts (dates, deadlines, multi-session items). NOT echoes of priority queue.
   - **RECENT SESSIONS:** What happened. Max 3 rows.

**Size gate:**
1. **Count lines.** Target ≤100, hard max 130.
2. **If over max, prune in this order:**
   - RECENT SESSIONS >3 rows → remove oldest
   - FLAGS tagged more than a few sessions ago AND resolved → remove
   - Completed projects/domains → collapse to 1-line or remove
   - Detail tables → replace with 1-line + link to domain doc
3. **Read the HTML comment** at the top of session-context.md for the full rules.
4. **If still over 130 after pruning** → flag to Jordan.

### 4. PERSIST SILENT-TRACKED INSIGHTS

Before writing session-context.md, classify each silently-tracked insight from this session (decisions, trade-offs, dead ends, patterns, near-misses) and route it:

| Insight type | Route to |
|---|---|
| Maps to an existing tracker row | Update that row's status/notes in place |
| New tracker-worthy item | Add row to the relevant tracker doc (e.g., `CC-DEVELOPMENT-PLAYBOOK.md`, domain roadmaps, or `COO-REFERENCE-MANUAL.md`) |
| Skill / workflow / architecture correction | Update the specific doc directly |
| Cross-session learning with no obvious doc home | Surface to Jordan with a question: "[Insight]. Where should this land?" Wait for direction — do not persist to a default file. |
| Genuinely ephemeral (already acted on, no future value) | Surface verbally only |

**Default is persist.** Verbal-only is the fallback for genuinely ephemeral items, not the baseline. Insights evaporate if only surfaced in chat.

**Blog-idea forward capture (D1/D5):** If any session insight is blog-shaped (failure story, pattern, teardown), append a candidate row to `AGENCY/Marketing/BLOG-IDEA-BACKLOG.md` (format: idea | source file+section | supporting artifacts | angle | status=open). This is idea-mining only — never draft the post itself; Jordan authors every word per D1/D5.

**Additive-candidate emission (CC-config loop):** If a persist-bound insight is a *systemic/cross-cutting* additive reference for a config-surface doc (a new reference note that would be appended to e.g. `CC-DEVELOPMENT-PLAYBOOK.md`), do NOT inline-add it here — route it to **Step 5.6** to emit as a loop candidate instead. Session-local references stay inline-added per the table above. **Emit XOR inline-add per item, never both** (double-apply guard).

### 4.5. CASCADE PROMPT SWEEP (MODEL-ROUTING-PROTOCOL Rule 12 enforcement)

Per `HAIOS/Knowledge/MODEL-ROUTING-PROTOCOL.md` v1.2 §Doctrine Integration build-queue item. Closes the gap where mechanical work leaks into Opus sessions because no skill step explicitly checks for it.

Two halves — backward audit + forward queue. Run both.

**Audit (backward — catch leaks):**
List every mechanical edit performed in THIS Opus session. Mechanical = pre-decided wording where a different model would produce identical output given identical inputs. Examples: SSOT-REGISTRY row adds, master index decision-log entries, path-ref fixes after moves/renames, session-context.md table updates (FLAGS / Doc versions / RECENT SESSIONS), version bumps with template-fill changelog entries.

For each:
- Was it done inline in Opus? Log it.
- **Apply the batch-size threshold** (`MODEL-ROUTING-PROTOCOL.md` v1.6 §Batch-size threshold) before flagging anything: the signal is **batch cost vs. session-handoff cost**, NOT edit count. Small incidental repairs surfaced while doing Opus work (stale paths, missing changelog rows, retired conventions) are **correctly** done inline — authoring a cascade prompt, closing Opus, opening Sonnet, executing, and closing again costs more than the fix. Do not flag these.
- Flag as a Rule 12 leak only when a **large deterministic batch** was executed inline that could have been queued whole — the founding pattern (~70% of an Opus session consumed by cascade tail work), not a handful of one-liners.
- Note: completed mechanical work is NOT retroactively migrated to a cascade prompt — the cost is already paid. The point of the audit is to surface the pattern so the next session queues earlier.

**Queue (forward — prevent future leak):**
Identify mechanical work that's READY (decisions locked this session) but doesn't HAVE to execute this session. Candidates: per-doc version bumps after a new doctrine, path-refs after a rename, registry entries for newly-created SSOTs, sibling-doc cascade tasks from a strategy arc.

**Authoring gate — test each task against CURRENT doctrine before it enters the prompt** (`MODEL-ROUTING-PROTOCOL.md` v1.9 §Authoring gate). "Decisions locked this session" is not the same as "still doctrine-legal." A task list inherits dispositions that were correct *when made*; doctrine keeps moving, and a cascade is written once. **There is no downstream gate** — §Sonnet session entry forbids Sonnet from improvising or ruling on scope, so Sonnet structurally *cannot* catch an anti-doctrine task. Authoring is the only gate a cascade has. Per task: *is this doctrine-legal today, or only on the day it was decided?* (For example, a cascade queued a dead-venture sweep that Rule 14/D12 had forbidden since then — struck at execution, two sessions late, at Opus cost.)

For each:
- **Arc-in-progress** (≥1 sub-strategy still to draft, or cascade not yet executed): APPEND to the running arc cascade prompt at `HAIOS/Sonnet-Cascade/S{first-session}-{arc-topic}-cascade-completion.md` per MODEL-ROUTING-PROTOCOL §Running cascade prompt. Bump prompt version + add task block with originating session number.
- **Standalone cascade need** (no active arc, but cascade-worthy work exists): create `HAIOS/Sonnet-Cascade/S{N}-{topic}-cascade-completion.md` with full task spec (context, numbered tasks with `oldText`/`newText` blocks where decidable, verification checklist, exit criteria, Sonnet-session notes).
- **Neither** (no arc, no standalone need): skip silently.

**Cascade prompt task format (per MODEL-ROUTING-PROTOCOL):**
```yaml
---
status: ACTIVE  # | PAUSED — needs Opus adjudication | COMPLETE
originating_sessions: [...]
targets: [list of files this prompt will modify]
---
```
+ numbered tasks with exact edits + verification reads + exit criteria.

**`COMPLETE` is NOT the end state — `ARCHIVED` is** (`MODEL-ROUTING-PROTOCOL.md` v1.9 §Terminal state — Jordan ruling). The executing Sonnet session marks `status: COMPLETE` in frontmatter and **then moves the file to `HAIOS/Sonnet-Cascade/Archive/`**. The move is the completion; the status field is not. So when authoring the prompt: do NOT write an exit criterion that stops at COMPLETE, and do NOT offer "whether to archive" as a choice — the terminal state is not a per-cascade variable. A cascade left in the active directory is a **re-execution hazard**, and cascade tasks are routinely non-idempotent (file moves, appends, registry inserts do not survive a second run). **Status is a claim; location is a fact.** (For example, a cascade was executed, then left `status: ACTIVE` in the active dir with non-idempotent moves at Tasks 3/4 — only a reader's attention stood between it and a second execution.)

**Skip conditions:**
- Audit: skip if no mechanical edits performed this session (rare).
- Queue: skip if no arc-in-progress AND no standalone cascade-worthy work.

**Output line for close report:** `Cascade sweep: ✅ no leak / no queue` │ `⚠️ [N] inline mechanical edits — [brief]` │ `📋 cascade prompt updated at [path]` │ combination of the above.

*evidence (first documented audit, retroactive):* Opus performed 4 mechanical edits inline this session — SSOT-REGISTRY 2-row add, master BUILD-ROADMAP decision-log row, BUSINESS-SYSTEM-MAP path-ref fix after rename, session-context.md FLAGS + Doc versions + RECENT SESSIONS rewrite. Aggregate ~13–15k Opus tokens. Each rationalized as "too small for cascade overhead"; aggregate is exactly the leak Rule 12 was codified to prevent. Pattern surfaced this session as the trigger to add Step 4.5.

### 5. DRIFT AUDIT

Catch four failure modes before writing session-context.md: stale tracker rows, mismatched skill triggers, chat-only structured outputs, and a stale derived AP index. All are failures where execution completed but secondary bookkeeping or artifact persistence was missed.

**Tracker drift check (#45):** For each NEXT SESSION item executed this session that references `Tracker #N` (in any tracker doc — `CC-DEVELOPMENT-PLAYBOOK.md`, `SKILLS-ROADMAP.md`, domain roadmaps, etc.):
1. Open the originating tracker
2. Verify row #N shows `applied` / `resolved` — not `proposed`
3. If still `proposed` after this session's work → backfill status immediately, or flag to Jordan if status is unclear

*evidence:* Tracker #29 (skill reachability audit) executed but its tracker row stayed `proposed` for many sessions until caught during later cross-reference work. Mechanism: session-end wrote session-context.md correctly but didn't walk back to the originating tracker.

**Trigger drift check (#25):** If any skill under `AOS/Skills/` was added, modified, or deprecated this session:
1. Read the skill's `description` field (YAML frontmatter)
2. Compare against the trigger row for that skill in `claude-instructions.md` (the Skills table)
3. Verify triggers match realistic user phrasings — not just skill-internal wording
4. If mismatch → update the trigger row in `claude-instructions.md`; this also cascades to userPreferences sync (step 7 cascade table)

*evidence:* `skill-creator-skill` description included "packaging skills" but the userPreferences trigger said only "creating/updating skills". Jordan said "package the cc skill" → no match → COO improvised. Fix added "packaging skills for install" to the trigger row.

**Persist drift check (#49) — REWRITTEN. The trigger is the REQUEST, not the output shape.** Per `HAIOS/claude-instructions.md` v25.40 § ENFORCEMENT: What Gets Written to Drive. This check now runs in **both directions**, and the second direction is the one that fires more often:

**(a) Should-have-persisted.** For each thing Jordan **asked for as a deliverable** (he requested a doc, named an output path, or it is an ongoing artifact he will reopen): verify a file was written to `MRMINOR/` this session via `Filesystem:write_file`. If not → write it now (invoke `doc-management-skill` PRE-FLIGHT for placement).

**(b) Should NOT have been created.** For each file COO wrote to `MRMINOR/` this session: verify Jordan actually asked for it. **A structured chat answer is not a request for a document.** If Jordan asked to *see* a review, analysis, plan, or audit and COO produced a file instead → flag it, name the file in the session-end report, and ask Jordan whether to delete. Do not quietly leave it.

**Do NOT flag a chat-only response merely because it had headers, tables, or length.** Under v25.40 that is correct behavior, not a violation. The old version of this check tested output shape and would now flag correct answers every session.

*evidence (direction a):* COO produced a multi-section analysis as chat-only; Jordan escalated; persisted after the fact. *evidence (direction b, and the reason this check was inverted):* Jordan asked COO to **display** an open-items review in chat; COO wrote `HAIOS/OPEN-ITEMS-REVIEW.md` instead. Jordan deleted it and said he was afraid to look in the filesystem. The old rule did not merely permit that file — its output-shape trigger with `default = YES` **required** it. One-directional enforcement produced one-directional failure.

**Completion-sync drift check (#50):** Backstop for doc-management-skill v2.11 §Completion sync + §Cascade-pre-staged completion sync (both atomic-with-verification).

*Primary path (CC-execution):* For each NEXT SESSION item executed this session that referenced a build-roadmap row:
1. List items where COO verified CC (or COO-direct) completion this session (per Quick Rule 3 sequence: CC reports, COO verifies, Jordan reviews).
2. For each, open the referenced roadmap row in the target file (AGENCY-BUILD-ROADMAP.md, BUILD-ROADMAP.md, BUSINESS-OPS-BUILD-ROADMAP.md, APP-FACTORY-BUILD-ROADMAP.md as relevant).
3. Verify the row shows `status: DONE` + completion session ref + verification note.
4. If status is still ACTIVE/IN PROGRESS but the work landed and was verified this session → flag as completion-sync drift; backfill the roadmap update before closing the session.

*Cascade path (Sonnet-cascade-execution):* For each cascade-completion file written or finalized this session:
1. Read the cascade-completion file at `HAIOS/Sonnet-Cascade/S{N}-{topic}-cascade-completion.md`.
2. If `status: COMPLETE`: enumerate any constituent roadmap-update items (e.g., "Edit AGENCY-BUILD-ROADMAP.md row P1-MKT-04 → status: DONE").
3. For each, open the referenced roadmap row and verify `status: DONE` + cascade-execution session ref + cascade-completion path as verification trail.
4. If status is still ACTIVE/IN PROGRESS while cascade-completion claims `status: COMPLETE` → flag as cascade-completion-sync drift; backfill the roadmap row before closing the session.
5. If `status: PARTIAL`: cross-check per-item disposition (DONE / SKIPPED / FAILED-reason) against roadmap state — DONE items must be propagated; SKIPPED/FAILED items must remain at prior status (not DONE).

*evidence:* codified the CC-execution path pre-emptively (Jordan question about build-roadmap freshness from completions). extended to cascade-pre-staged work — doc-management-skill v2.10 §Completion sync covered CC-completion but not cascade-pre-staged roadmap updates; v2.11 added §Cascade-pre-staged completion sync, this drift check extended in lockstep. Atomic completion-sync per doc-management-skill v2.11 §Completion sync + §Cascade-pre-staged completion sync is the primary defense; this compound drift check is the backstop.

**AP-index staleness check (#51 — D-13 half 2; ROUTING RULE replaces capability claim):** `ANTI-PATTERNS.md` carries a generated index in a sentinel-delimited region. It goes stale the instant any AP row is appended, and a stale index makes new APs unreachable to every index-based reader. `process-feedback` v1.9 regenerates on the CC append path; **this check covers the paths CC does not own** — direct COO edits and Sonnet cascades.

**This check has encoded a capability claim twice and been wrong both times.** v1.19: "COO cannot execute" — false, it generalized two tool-specific misses without checking the third. v1.20: "COO CAN execute via Desktop Commander" — true at that time, false by that session, when DC went persistently disconnected in this environment. A capability claim is a claim with a shelf life, but it reads as physics and so never gets retested. **The claim is replaced below by a routing rule that names the actor per phase instead of the tool per environment.**

**Detection — always available, no tool preconditions.**
1. `Filesystem:copy_file_user_to_claude` → `HAIOS\Knowledge\BestPractices\ANTI-PATTERNS.md`. Lands at `/mnt/user-data/uploads/ANTI-PATTERNS.md`.
2. In `bash_tool`, run the file's own **LAW 3** command twice — once scoped to the sentinel region, once against the whole file:
```bash
cd /mnt/user-data/uploads
# index max (inside the generated region)
sed -n '/AP-INDEX:BEGIN/,/AP-INDEX:END/p' ANTI-PATTERNS.md | grep -oE '^\| *[0-9]+' | grep -oE '[0-9]+' | sort -n | tail -1
# file max (live append tail)
grep -oE '^\| *[0-9]+' ANTI-PATTERNS.md | grep -oE '[0-9]+' | sort -n | tail -1
```
3. Equal → clean. **Mismatch → the index is stale by (file max − index max) APs.** Surface it and route regeneration per below.

Use LAW 3's command, never a hand-rolled regex: it is suffix-tolerant by design (leading digits only), and rows are appended chronologically, so the last row is not the max. Verified twice against the live 743-line file — index max = file max = 276 both runs.

**Regeneration — never COO, regardless of tooling.** `--write` mutates a canonical under `the shared drive`. Route to CC (prompt) or Jordan:
```
py "HAIOS\Tools\ap-index\generate_ap_index.py" --write
```
then re-run `--check` to prove exit 0. The generator is the authoritative instrument for regeneration — it applies its own region rule and dedup. **Never hand-edit the sentinel region to "fix" it**; the region is generated, never hand-maintained.

**Desktop Commander is an optimization, never a prerequisite.** When DC is connected, `start_process` running `cd "HAIOS\Tools\ap-index"; py generate_ap_index.py --check` is faster and catches line-ending drift the grep read is structurally blind to. When DC is absent — the standing state since then — detection proceeds unchanged via copy+bash. **DC's presence changes the speed of this check, never whether it runs.**

*evidence:* CC appended during that session's site build without regenerating. Index max sat at #261, file max was #264 — 3 APs unreachable, coverage 100% → 98.6%, **one session after R-1 shipped the index at 100%**. Caught by exactly this two-grep read, at no cost, before anything consumed the stale index. Measured decay rate is **one session**, which is why the CC-path `--write` alone is insufficient: it leaves the COO and cascade append doors open.

**Skip conditions (per check):**
- Tracker drift: skip if no tracker-referencing items were executed this session.
- Trigger drift: skip if no skills under `AOS/Skills/` were added/modified/deprecated this session.
- Persist drift: **never skip** — every session is scanned (this is the safety net).
- Completion-sync drift: skip if NO roadmap-touching completions this session — neither NEXT SESSION items verified-complete (CC path) NOR cascade-completion files written/finalized (cascade path). Run path-specific check only when that path has activity.
- AP-index staleness: **never skip** — detection is copy+bash and has no tool preconditions, so there is no environment in which this check is unavailable. The failure is silent by construction. A session that never touched `ANTI-PATTERNS.md` can still be the session that discovers a prior session's stale index.
- Discovery week debrief: skip if no `discovery week LIVE` in FLAGS.

**Discovery week debrief check:** If session-context FLAGS shows a `discovery week LIVE`, verify today’s `DAY-N-YYYY-MM-DD.md` was written or appended this session. If Jordan dropped field notes but the file was not updated — flag before closing. Non-skippable per ops playbook §7.3.

Output line for report: `Drift audit: ✅ clean` OR `⚠️ [N items] — [brief summary]` — covers all five checks (tracker, trigger, persist, completion-sync, AP-index staleness).

### 5.6. ADDITIVE CANDIDATE EMISSION (CC-config loop — Stream A)

Feeds the CC-config self-improvement loop one additive candidate class: **reference-add** (a new, additive reference note for a config-surface doc). The loop classifies + gates + auto-applies the AUTO-safe class; session-end only EMITS the raw candidate — it never sets `gate_class` (spine invariant: source emits, loop classifies).

**Scope (deliberately narrow):** emit **reference-add only**. The apply path (`apply.py`) is **append-only** — stale-ref (in-place replace) and token-bloat (trim) are NOT yet loop-applicable, so keep inline-fixing those under Steps 7–8 as today. When the apply path gains replace/trim, widen this step. (`HAIOS/Architecture/CC-CONFIG-LOOP-PHASE-1-BUILD-PLAN.md` G1 + 4b-emit.)

**When to emit (lean-(i) selection):** only for a *systemic / cross-cutting* additive reference — a standing config-surface note worth routing through the loop. **Session-local** references (clearly belonging to one doc, decided this session) stay **inline-added** per Step 4. **Emit XOR inline-add per item — never both** (an inline-add carries no sentinel, so the loop would append a second copy → double-apply). Most sessions emit nothing; skip silently when there is no systemic reference-add.

**Eligibility gate (all must hold):**
1. Target is a **config-surface reference doc** (per `HAIOS/Architecture/CONFIG-SURFACE-INVENTORY.md` §3 — e.g. `CC-DEVELOPMENT-PLAYBOOK.md`). Not a rule/hook/skill/CLAUDE.md (those are behavior-gating → SURFACE, not emittable here).
2. The change is an **append** of an **additive, reversible** note. No in-place edits.
3. Content is **true** and **non-behavior-gating** (reference knowledge, not an enforced rule).

**How to emit:** append one entry to `AOS/CC-Config/additive-candidates-inbox.md` under `## Entries`, in the EXACT format the drain (`ingest_stream_a.py`) parses:

````
### ADD-S{N}-{NN} [status: pending]

```json
{
  "id": "ADD-S{N}-{NN}",
  "drift_type": "reference-add",
  "candidate_type": "reference-update",
  "target_display": "HAIOS/Knowledge/BestPractices/CC-DEVELOPMENT-PLAYBOOK.md",
  "target_fs_path": "HAIOS\\Knowledge\\BestPractices\\CC-DEVELOPMENT-PLAYBOOK.md",
  "edit_kind": "append",
  "sentinel_start": "<!-- CC-CONFIG-LOOP-ADD-S{N}-{NN}-START -->",
  "sentinel_end": "<!-- CC-CONFIG-LOOP-ADD-S{N}-{NN}-END -->",
  "content": "### {section title}\n{additive reference body, with newlines escaped as \\n}",
  "session": {N},
  "rationale": "{why this is a systemic reference worth routing through the loop}"
}
```
````

**Format invariants (the drain is strict):**
- Header `### {id} [status: pending]` — `id` has **no spaces** (drain regex `\S+`). Convention `ADD-S{N}-{NN}` (zero-padded, unique per entry).
- The `id` inside the JSON MUST equal the header `id` (it is the dedup key).
- `edit_kind` MUST be `"append"` (the only AUTO-eligible kind; `apply.py` SURFACE-skips anything else).
- `sentinel_start`/`sentinel_end` are unique HTML comments; apply wraps content as `\n{start}\n{content}\n{end}\n` and gates on sentinel absence (idempotent re-apply guard).
- `target_fs_path` = full Windows path (double-backslash in JSON); `target_display` = repo-relative.
- `content` = the raw markdown to append, with `\n` escaped.

After emitting, the entry sits `pending` until the loop drains it (flips to `consumed`). Do NOT also inline-add the same content.

**Install note:** this step is canonical at `AOS/Skills/session-end-skill/`; it is live only after Jordan installs the updated skill in Claude Desktop (tracked under “Skill installs pending”).

**Skip condition:** skip silently unless this session produced a systemic config-surface reference-add deliberately routed to the loop (the common case).

### 6. UPDATE SESSION-CONTEXT.MD
Read, update, write:
```
Filesystem:read_file → HAIOS\session-context.md
```

**Content Gate:** session-context.md = current state ONLY.
- ✅ State: session number, focus, status, flags, pending skill installs
- ✅ Size: ≤100 lines target, 130 hard max
- ❌ Rules → `claude-instructions.md`
- ❌ How-to → skills
- ❌ Architecture → design docs
- ❌ Domain detail tables → domain docs (link instead)
- ❌ Reference data that rarely changes → 1-2 lines max in REFERENCE section

If adding content that isn't state, put it in the right place instead.

Update sections (each item in ONE place only — see step 3 dedup rules):
- **Session number:** Increment
- **Last Updated:** Pointer only — `S{N} — see RECENT SESSIONS table below for detail`. Do NOT restate the session narrative here; that content belongs in the RECENT SESSIONS row (step below) and nowhere else.
- **NEXT SESSION:** Actions for next session. Tag each with [O] [S] or [H] per model tier convention in RULES comment. No status context.
- **NEXT SESSION → Skill installs pending** (subsection — **COO skills only** scope): If any `AOS/Skills/[skill]/` file was edited this session, add a line per skill: `` `[skill-name]` — brief change summary (Sxxx). Canonical: `AOS/Skills/[skill-name]/` ``. Jordan installs from canonical path on Claude Desktop. Remove entries when Jordan confirms install in a future session. **CC-operational skills (6 under `~/.claude/skills/`) do NOT use this staging** — they propagate to the Drive mirror via the step 8.5 sync instead. See `references/LAYER-TAXONOMY.md` Skill Placement.
- **WORKSTREAMS:** Status changes only. Include cross-stream impacts from step 2.
- **RECENT SESSIONS:** Add current row. Remove oldest if >3 rows.
- **FLAGS:** Only persistent time-sensitive alerts. Remove resolved flags more than a few sessions old.

```
Filesystem:write_file → HAIOS\session-context.md
```
**Tool is fixed by path, not preference (Quick Rule 5).** `MRMINOR/` → **Filesystem MCP only** — never container `create_file`, never `Desktop Commander:write_file`. (DC in that tree = content grep only.) This step previously specified `Desktop Commander:write_file` against an `MRMINOR/` path, contradicting Rule 5; corrected on sight.

### 7. CASCADE CHECK
For each file changed this session, check if downstream docs need updating:

**Process:**
1. List all files created or modified this session
2. For files that have **Cascade Rules** sections, scan the listed targets
3. For each target, verify: was it already updated this session?
4. Flag any missed cascades

**Common cascade targets:**
| When you change... | Also check... |
|---|---|
| DB schema changes | DATABASE-DOMAIN.md (archived — live DB is authoritative — live DB is authoritative) |
| BUILD-ROADMAP.md (master index) | session-context.md (next priorities), domain roadmaps if phase gates shifted |
| Domain roadmaps (*-BUILD-ROADMAP.md) | Master index dependency map (if cross-domain deps changed), session-context.md |
| claude-instructions.md | session-context.md (version), prompt Jordan to copy to userPreferences |
| SKILLS-ROADMAP.md | claude-instructions.md (skill count/table) |
| `AOS/Skills/[skill]/` (any skill source edit) | session-context.md (add to "Skill installs pending"), prompt Jordan to install |
| AGENCY-KNOWLEDGE-ARCHITECTURE.md | Master index, SERVICE-CATALOG.md |
| AGENCY-STRATEGY.md | AGENCY-BUILD-ROADMAP.md, AGENCY-OPERATIONS.md |
| New docs created | SSOT-REGISTRY-MASTER.md (register them) |
| Build items completed | Relevant domain roadmap (mark done) |
| Doc with Quick Reference / TL;DR / summary block edited | Verify summary block against body for drift (stale thresholds, outdated numbers, renamed tools). Also: frontmatter `version` + changelog entry reflect THIS session's work. |

**If missed cascades found:**
- Update them now before closing
- Include in the changed files list

### 8. COLLECT CHANGED FILES + CC ARTIFACT SWEEP
List all files created or modified this session:
- MRMINOR/ docs (*.md)
- Skills (if any installed)
- Exclude: temporary files, non-indexed paths

**CC artifact orphan sweep:** If any CC prompts were dispatched this session, list the directory where each prompt was written and verify no `CC-*` files remain (prompt, feedback, build log, feedback-processed). CC's `process-feedback` skill sometimes claims to archive but leaves files in place. This is the safety net — `cc-prompt-skill` Step 5 should catch orphans at processing time, but session-end catches anything that slipped through.

If orphans found → move to the ONE flat `MRMINOR/Archive` with a self-locating name: `S{session}-{verb}-{subject}-{type}.md` (per `doc-management-skill` §Session artifact routing + `HAIOS/Knowledge/ARCHIVE-ARCHITECTURE-DECISION.md`). **Never `Archive/CC-Prompts/`** — that convention was retired.

**Archive-move broken-ref sweep:** When any file is moved to `Archive/` this session, grep each in-scope doc for the old path. Flag any doc that still cites the pre-archive path as a broken-ref candidate for Phase 5 or next doc-audit cycle. Pattern: two archive moves (DOC-AUDIT-PATTERNS-CAPTURE.md, C1-POST-MORTEM.md) each accumulated 3-4 stale refs across source docs before Phase 3 audit caught them. A one-step grep at archive time prevents multi-session accumulation.

Grep pattern (run per archived filename):
```bash
grep -rn "HAIOS/Architecture/{archived-filename}" "HAIOS/"
```

For each hit: either (a) inline-fix the reference to point to the new `Archive/` path if the fix is unambiguous, or (b) note the stale reference in the session report + flag for next doc audit's Phase 3 staleness pass. Historical-record exceptions (lines that say "moved X from A to B") are preserved.

### 8.5. CC-CONFIG SYNC TRIGGER

CC's instance config — `~/.claude/CLAUDE.md`, `rules/`, `settings.json`, `hooks/`, `skills/`, `agents/`, `commands/` — is canonical on Desktop. The Drive mirror at `AOS/CC-Config/` is refreshed by `sync-cc-config.ps1` (robocopy `/MIR`, one-way Desktop→Drive). Session-end is the chosen sync cadence per `HAIOS/Architecture/CANONICAL-GAP-DECISION.md` (approved).

**Why this step exists:** Without a session-end trigger, the Drive mirror goes stale. Stale mirror → stale semantic-index coverage of CC skills → CC audit search misses recent edits. Also breaks the restore path if Jordan ever needs to rebuild `~/.claude/` from Drive.

**What COO does:**
1. Scan this session for any `~/.claude/*` edits (CC skill SKILL.md, references/, rules/, hooks/, CLAUDE.md, settings.json). Note whether edits occurred — informational only, don't gate on this; the sync should run regardless because CC may have self-edited between sessions.
2. Include the Jordan handoff line in step 9 REPORT COMPLETION output (see format below).

**What Jordan does:** Run the sync before closing:
```powershell
& "AOS\CC-Config\run-sync.bat"
```
Or PowerShell direct:
```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force; & "AOS\CC-Config\sync-cc-config.ps1"; echo "Exit: $LASTEXITCODE"
```
Silent on success. Exit 0 = good. Errors log to `AOS\CC-Config\sync-errors.log`.

**Note:** "COO cannot execute PowerShell directly" was never the reason for this handoff — that claim was false when written (disproved via `Desktop Commander:start_process`, though DC has been disconnected since then; see check #51 for why capability claims are no longer encoded in this skill). **This stays a Jordan handoff on AUTHORITY, not capability:** `sync-cc-config.ps1` runs robocopy `/MIR`, which is a destructive one-way mirror that deletes Drive-side content to match Desktop — a Tier 1 system change per the decision matrix, and W55 documents the exact case where `/MIR` against a missing canonical compounds a loss into data destruction. **"COO may not" is the correct constraint; "COO cannot" was never true and hid the distinction.** If Jordan skips a given session-end, the next session-end catches up (`/MIR` is idempotent, mirrors current state). If this becomes a repeated miss pattern, consider adding scheduled-task fallback (originally rejected as hybrid option per that session decision).

### 9. REPORT COMPLETION
```
✅ Session [N] closed
- Context saved: session-context.md
- Files changed: [list with paths]
- Insights persisted: ✅ [count routed] | ⚠️ [any unresolved]
- Cascade sweep: ✅ no leak / no queue | ⚠️ [N inline leaks] | 📋 cascade prompt at [path] | combination
- Drift audit: ✅ clean | ⚠️ [N items found and fixed]
- Pending skill installs (COO): ✅ none | ⚠️ [count] — see session-context NEXT SESSION
- Cross-stream impacts: [any noted, or "none"]
- Roadmap dependency check: ✅ no roadmap changes | ✅ deps verified | ⚠️ [deps updated]
- Cascade check: ✅ all targets updated | ⚠️ [missed items fixed]
- **Next session model: [Sonnet | Opus]** — [reason based on NEXT SESSION tag distribution]

⚠️ **Jordan action before closing:** run CC-Config sync —
`& "AOS\CC-Config\run-sync.bat"` (silent on success, exit 0 = good)

Orchestrator note: Changed files may need semantic index update.
```

## Handoff Contract

When complete, state:
- Session number closed
- session-context.md updated: ✅
- Insights persisted: ✅ [count] | ⚠️ [unresolved]
- Cascade sweep: ✅ no leak / no queue | ⚠️ [N leaks] | 📋 cascade prompt at [path]
- Drift audit: ✅ clean | ⚠️ [N items fixed]
- Pending skill installs (COO): ✅ none | ⚠️ [count + list]
- CC-Config sync handoff issued to Jordan: ✅ (run-sync.bat command in report)
- Dedup check: ✅ or ⚠️ [items that were duplicated]
- Size gate passed: ✅ [line count] or ⚠️ [issue]
- Cross-stream impacts: [list or "none"]
- Roadmap dependency check: ✅ or ⚠️
- Cascade check: ✅ or ⚠️ [what was missed and fixed]
- CC artifact sweep: ✅ clean | ⚠️ [orphans found and archived]
- Files changed this session: [list]

Orchestrator decides next action (e.g., semantic index update).

## Dependencies

- Required: `Filesystem:read_text_file` + `Filesystem:write_file` (all `MRMINOR/` reads + writes — Rule 5). Desktop Commander is for `~/.claude/` edits and content grep only.
- Related: `semantic-index-update-skill` (orchestrator may invoke after)

## Authority

Tier 3 (Autonomous) — session context is operational data

## Error Handling

| Issue | Action |
|---|---|
| session-context.md not found | Ask Jordan to verify path |
| Write fails | Check if file is open elsewhere |
| Over 130 lines after pruning | Flag to Jordan for structural review |
| Cascade target file not found | Note it, don't block session end |

## Notes

> Session insights logged to session-context.md silently per session-end-skill Step 4. JOURNEY-CAPTURE.md retired.
- **Size discipline:** session-context.md bloats naturally. Dedup + size gate are the preventative controls; enforce every session.
- **Duplication is the #1 bloat cause.** Same item in NEXT SESSION + WORKSTREAMS + FLAGS = 3x the lines for 1x the info. The dedup test (step 3) prevents this.
- **Cascade check catches doc drift.** Most docs have Cascade Rules sections. The common targets table (step 7) covers the 80% case. When in doubt, read the Cascade Rules section of the changed file.
- **Roadmap structure:** Build roadmap is decomposed into master index + 5 domain roadmaps. When roadmap items are touched, always check cross-domain dependencies in the master index at `HAIOS/Architecture/BUILD-ROADMAP.md`.
- **Skill storage doctrine (amended):** Two tracks. **COO skills** canonical at `AOS/Skills/[skill-name]/` — Jordan installs to Desktop `/mnt/skills/user/` from canonical. Staging: "Skill installs pending" subsection. **CC-operational skills** canonical at `~/.claude/skills/[skill-name]/` on Desktop — Drive mirror at `AOS/CC-Config/skills/` via step 8.5 sync. No staging (CC is live on edit). Full doctrine: `AOS/Skills/doc-management-skill/references/LAYER-TAXONOMY.md` Skill Placement. Amendment memo: `HAIOS/Architecture/CANONICAL-GAP-DECISION.md`.
- **Doctrine: new task = new session**. Closing cleanly here is how this doctrine is enforced — session-context.md is the handoff mechanism, not carried context. The next task starts fresh in a new session, even if the current one has headroom. Full guidance: `CC-DEVELOPMENT-PLAYBOOK.md` § Context Management.
- **Drift audit:** Step 5 catches four failure modes where execution succeeded but secondary records or artifacts drifted: tracker row status (#45), skill trigger phrasing (#25), chat-only structured outputs (#49, Persist-Before-Present violations), and completed-but-not-marked-DONE roadmap rows (#50, completion-sync drift — CC-execution and cascade-pre-staged variants). #45 and #25 emerged n=1 from that session. #49 **rewritten to run in both directions** — the old version tested output *shape* and, under `claude-instructions.md` v25.40 (trigger = the request, not the shape), would have flagged correct chat answers as violations every session. Direction (b), unrequested files, is now the more common failure: produced a doc when Jordan asked for a chat display, because the old rule's `default = YES` required it. #50 codified pre-emptively (Jordan question about build-roadmap freshness); cascade path added (gap between CC-completion and cascade-pre-staged work). Primary defense is doc-management-skill v2.11 §Completion sync + §Cascade-pre-staged completion sync (atomic-with-verification); this is the backstop. Compound check keeps the cost low.

## Changelog

| Version | Changes |
|---|---|
| 1.23 | **Check #51's capability claim replaced by a routing rule — the claim had been encoded twice and was wrong both times.** v1.19 said "COO cannot execute" (false: generalized two tool-specific misses without checking the third); v1.20 said "COO CAN execute via Desktop Commander" (true at that time, false by that session when DC went persistently disconnected). **Both failures are the same failure**: a claim about which tool is present, written into doctrine where it reads as physics and never gets retested. The correction is not a third, better claim — it is to stop making the claim. **#51 now names the actor per phase, not the tool per environment.** Detection = `copy_file_user_to_claude` → `/mnt/user-data/uploads/` → the file's own LAW 3 command in `bash_tool`, run twice (sentinel region, then whole file); this bridge has no tool preconditions, so "never skip" is now structurally true rather than aspirational. Proven and re-run on the live 743-line file (index max = file max = 276 both times). Regeneration (`--write` against a `the shared drive` canonical) routes to CC or Jordan **on authority, unchanged** — the old text bundled it to capability, which is why losing DC looked like losing the check. **Desktop Commander demoted from prerequisite to optimization:** when connected, `--check` via `start_process` is faster and catches line-ending drift the grep read is blind to; when absent, detection is unaffected. Skip-conditions bullet and the §8.5 sync-handoff cross-reference updated in lockstep — §8.5's handoff was already correctly grounded in authority (`/MIR` is a destructive Tier 1 mirror; W55), it just cited #51 for a capability proof that no longer lives there. |
| 1.22 | **"Last Updated" field converted to a pointer — it was silently duplicating RECENT SESSIONS.** Step 3's dedup test and Step 6's section-update instructions both gained an explicit rule: `Last Updated` = `S{N} — see RECENT SESSIONS table below for detail`, never a restatement. Evidence: by that session the field had grown into a full paragraph re-narrating the RECENT SESSIONS rows verbatim — same content, two formats, growing every session because Last Updated has no size cap or prune rule the way RECENT SESSIONS does (max 3 rows). Caught by Jordan asking "anything duplicated?" post-close, not by this skill's own dedup test, which never covered a field outside its four named sections. Jordan-directed fix, same session. |
| 1.21 | **Drift check #49 inverted — it was about to start flagging correct behavior.** `claude-instructions.md` v25.40 replaced the output-shape trigger (≥2 headers/tables → persist, default YES) with a request trigger (Jordan asked for a deliverable → persist; otherwise answer in chat). #49 still encoded the old shape test, so from v25.40 onward every structured chat answer — now the *correct* output — would have been logged as a Persist-Before-Present violation, and the retroactive-write step would have converted correct answers into exactly the unrequested files v25.40 bans. **Now bidirectional:** (a) requested deliverables that were not persisted; (b) **files created that Jordan never asked for** — the direction that had no check at all. evidence for (b): Jordan asked COO to display an open-items review in chat, COO wrote it to Drive, Jordan deleted it and said he was afraid to look in the filesystem. The rule did not merely permit that file, it required it. Notes block updated to match. |
| 1.20 | **Two false capability claims removed — COO can execute, and has been able to since Desktop Commander was connected.** Check #51 said *"Detection only, not regeneration: COO cannot execute the generator (no `the shared drive` from container bash, no execute on Filesystem MCP)"* and §8.5 said *"COO cannot execute PowerShell directly — this is a mandatory Jordan handoff."* **Both cited premises are true; both conclusions are false.** `bash_tool` does run on Claude's computer with no `the shared drive`, and Filesystem MCP does lack execute — but `Desktop Commander:start_process` runs on Jordan's machine and has both. Two tool-specific misses were generalized into a capability claim about the actor, and no one checked the third tool. It held for many sessions and shrank the COO action space the whole time. **Disproved by execution, three times:** `generate_ap_index.py --check` and `check_ap_cells.py --check` (exit 0 each, run after every edit in an 11-edit `ANTI-PATTERNS.md` pass) plus a PowerShell byte-level line-ending inspection that caught an LF flip `--check` is structurally blind to. **Same mechanism as that session's sub-case, one layer up:** there, a single targeted path check proved "not at this path," not "does not exist anywhere"; here, two tool probes proved "not via these tools," not "not at all." A negative capability claim needs the same breadth of probe as a negative existence claim, and it is more expensive to get wrong — a phantom file wastes a search, a phantom incapacity permanently narrows what the actor attempts. **#51 rewritten:** run the generator's own `--check` via DC (authoritative — it applies its own region rule and dedup, which two greps cannot), `--write` + re-`--check` on staleness; the two-grep read demoted to a DC-unavailable fallback. **§8.5 rewritten:** the sync handoff **stays**, but on **authority, not capability** — `/MIR` is a destructive one-way mirror (Tier 1 system change; W55 is the case where `/MIR` against a missing canonical turns a loss into destruction). **"COO may not" is the correct constraint; "COO cannot" was never true and hid the distinction** — a capability claim standing in for an authority rule reads as physics and never gets retested. |
| 1.19 | **Step 5 gains drift check #51 — AP-index staleness, detection-only.** `ANTI-PATTERNS.md` carries a sentinel-delimited generated index (R-1 — 206 APs, coverage 38% → 100%, 127 previously write-only APs made reachable). **It went stale in one session:** CC appended during that session's site build without regenerating; index max sat at #261 against a file max of #264, dropping coverage to 98.6% and re-hiding 3 new APs. Measured decay rate = **1 session**. This is D-13 half 2 of 2 (`HAIOS/Architecture/GOVERNANCE-SUBSTRATE-AUDIT-FABLE.md` §5). Half 1 is `process-feedback` v1.9, which runs `generate_ap_index.py --write` on the CC append path — but CC owns only one of three append doors; direct COO edits and Sonnet cascades are uncovered, which is what this check catches. **Split by actor capability, deliberately: COO can detect but cannot regenerate** (no `the shared drive` from container bash, no execute on Filesystem MCP), so #51 is two greps comparing index-region max key to file max key, with regeneration queued to an actor that has execute. Never-skip: the failure is silent by construction and the check costs two reads. Existence proof: caught the staleness by exactly this read, at no cost, before any consumer hit the stale index. **Also fix-on-sight (`doc-management-skill` v2.18):** Step 6 + Dependencies specified `Desktop Commander:write_file` against `session-context.md`, an `MRMINOR/` path — direct contradiction of Quick Rule 5 (Rule 5: writes are routed by PATH; `MRMINOR/` is Filesystem-only, DC there is grep-only). Corrected to `Filesystem:write_file` with the routing rule stated inline. Every session-end since then either violated the skill or violated Rule 5. |
| 1.18 | **§4.5 reconciled with `MODEL-ROUTING-PROTOCOL` v1.9 — this skill was teaching a terminal state the protocol had just retired.** (1) **Cascade task format: `COMPLETE` is not the end state, `ARCHIVED` is** (v1.9 §Terminal state, Jordan ruling). The yaml block presented `ACTIVE | PAUSED | COMPLETE` as the full lifecycle and never mentioned the archive move, so a prompt authored from this skill could ship an exit criterion that stops at COMPLETE. Sonnet marks COMPLETE *then moves the file to* `Sonnet-Cascade/Archive/`; the move is the completion. Authors must not offer "whether to archive" as a choice — the terminal state is not a per-cascade variable. Rationale is the re-execution hazard, not tidiness: cascade tasks are routinely non-idempotent, and status is a claim while location is a fact (Note: executed, then left `status: ACTIVE` in the active dir with non-idempotent moves at Tasks 3/4). (2) **§4.5 Queue half gains the v1.9 §Authoring gate** — the Queue step *is* the cascade-authoring point, and "decisions locked this session" is not the same as "still doctrine-legal." A cascade is written once; doctrine moves. There is no downstream gate, because §Sonnet session entry correctly forbids Sonnet from improvising — so Sonnet structurally cannot catch an anti-doctrine task, and authoring is the only gate. (For example, a cascade queued a dead-venture sweep Rule 14/D12 had forbidden since then; struck at execution two sessions late, at Opus cost.) Referenced, not duplicated, per the instruction-migration principle — doctrine stays in MODEL-ROUTING. **Provenance:** the conflict was created by that session bumping MODEL-ROUTING to v1.9 and caught by this skill's own Step 7 cascade check, in the same session, before the protocol edit could go live unreconciled — the failure mode (three protocols disagreeing, every run forced to violate one) avoided by one session. |
| 1.17 | **Added Step 4 sub-step: blog-idea forward capture** (agency realignment cascade, task 1.9, D1/D5). Blog-shaped session insights (failure story / pattern / teardown) append a candidate row to `AGENCY/Marketing/BLOG-IDEA-BACKLOG.md` — idea-mining only, never AI-drafted prose. |
| 1.16 | Removed 1 stale ref to decommissioned venture (decommission cascade). |
| 1.15 | **Step 4.5 audit trigger corrected — the ≥2-inline-mechanical-edits threshold was measuring the wrong thing.** It fired on ~15 incidental stale-path repairs this session and reported a Rule 12 leak; Jordan overruled: a cascade's cost is prompt authoring + Opus close + Sonnet start + execution + close, which does not pay against a handful of one-liners. Threshold replaced with the batch-size test from `MODEL-ROUTING-PROTOCOL.md` v1.6 §Batch-size threshold — flag only when a *large deterministic batch* ran inline (the founding pattern: ~70% of an Opus session on cascade tail work), never on incidental repairs, which §Fix-on-sight requires be done inline anyway. Lockstep with MODEL-ROUTING-PROTOCOL v1.5 → v1.6. |
| 1.14 | **Fix-on-sight repair + changelog backfill** (`doc-management-skill` v2.18 §Fix-on-sight, Jordan standing instruction). (1) Step 8 CC-artifact orphan sweep routed to `Archive/CC-Prompts/` — a convention retired in favour of the ONE flat `MRMINOR/Archive` with self-locating names. Same stale-convention family as `quarterly-doc-audit-skill` §8 (repaired v2.5 this session): both skills still routed artifacts to per-type archive subfolders that no longer exist. Surfaced while executing this skill's own close sequence. (2) Backfilled the missing v1.13 changelog row (below) — the frontmatter was bumped to 1.13 at that time with no corresponding entry, so the version history claimed 1.12 while the file said 1.13. |
