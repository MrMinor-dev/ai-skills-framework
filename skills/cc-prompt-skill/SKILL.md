---
name: cc-prompt-skill
description: "Delegate tasks to CC. Triggers: CC prompts, CC feedback, delegating any execution task."
---

# CC Prompt Skill

**Version:** 1.38 | **Layer:** AOS

<!-- Changelog: §9 EXECUTION POLICY workflow-verification.md path corrected to project-level `.claude\rules\workflow-verification.md` (moved old `~/.claude/rules/` path was stale) -->

Write effective CC task prompts, delegate via file, capture feedback, improve continuously. This skill governs the COO↔CC interface for ALL task types — builds, fixes, tests, deploys, research, refactoring, migrations, content creation, audits.

## Quick Reference

```
COO writes prompt file → Jordan pastes one-liner → CC reads + plans + executes → CC writes build log + feedback file → COO absorbs learnings
```

**Jordan's dispatch (one message):**

`/goal {acceptance_condition}. Read the full task at "{filepath}" — use Plan Mode, then execute.`

`/goal` leads the message. It anchors CC to the completion condition before any file reading begins — the anchor must precede the file read, which the single message preserves by ordering, not by separation. See `CC-DEVELOPMENT-PLAYBOOK.md §/goal — Completion Condition Anchor` for full doctrine.

**Template v3 principle:** Prompts contain ONLY task-specific content. Repeating protocols (secrets, feedback) live in CC's global CLAUDE.md — CC reads them every session. The prompt template itself is split into a **router** (`references/PROMPT-TEMPLATE.md` — universal scaffolding + task-type index) and **task-type files** (`references/task-types/{TYPE}.md` — one per task type, spec + accumulated session learnings). COO reads router every prompt-write, then reads exactly one task-type file for the task at hand.

## Workflow

### 1. DETERMINE IF CC IS THE RIGHT LANE

**Delegation test:** Am I about to write executable code, CLI commands, scripts, or touch files on Jordan's machine?
- **Yes** → run the Storm-routing gate below, then continue to step 2.
- **No** (strategy, doc updates, analysis, webhook queries) → Handle directly. Stop here.

**Storm-routing gate (compaction-storm prevention).** Even when the delegation test says Yes, do NOT dispatch a REFACTOR / split / whole-file rewrite to CC when the target is **(a)** part of CC's permanent context tax (`CLAUDE.md`, `.claude/rules/*`, `settings.json`, large `SKILL.md`), or **(b)** >15 KB AND requires whole-file context. Loading such a file to rewrite it combines with system prompt + skills + permanent tax + thinking burn and crosses CC's auto-compact threshold mid-task → storm. These are COO-layer edits — handle directly, do not write a CC prompt. Surgical `old_str`/`new_str` edits, new-file creation, and small (<10 KB) files are fine for CC. Full routing table + variant defenses: `~/.claude/rules/compaction-storm-prevention.md` (global rule). Doctrine SSOT: `CONTEXT-ECONOMY-PLAYBOOK.md` § Routing table + `CC-DEVELOPMENT-PLAYBOOK.md` § Context Management → No CC Self-Refactor.

### 2. WRITE THE PROMPT FILE

Read `references/PROMPT-TEMPLATE.md` for the universal template.

**File conventions:**
- **Location:** Same folder as the relevant README or project root
- **Naming:** `CC-PROMPT-{VERB}-{SUBJECT}.md`
- **Verbs:** BUILD, FIX, TEST, DEPLOY, RESEARCH, REFACTOR, MIGRATE, CREATE, AUDIT
- **Examples:** `CC-PROMPT-BUILD-V3.md`, `CC-PROMPT-FIX-GMAIL-TRIGGER.md`, `CC-PROMPT-RESEARCH-AUTH-LIBRARIES.md`

**Prompt tier note:** This skill governs `CC-PROMPT-BUILD-*` and related one-time dispatch files — single execution, archived after use. A separate tier called **Per-Engagement CC Templates** (e.g., `A1-ASSESSMENT-CC-TEMPLATE.md`) is reusable across deliveries for the same SKU. Do NOT archive Per-Engagement CC Templates after each use — update them with delivery learnings.

**Critical rules:**
- **Self-contained.** CC has NO context from the COO session. Include everything: full file paths (never "the schema doc" or partial paths), endpoints, credentials file locations, expected outputs, verification steps.
- **Positive framing for style, negative for hard constraints.** Replace "don't over-explain" with "write concise, focused responses." Replace "don't try to do this in one response" with "spawn a specialist for each of frontend, backend, database." Reserve negative framing for hard constraints where the failure mode is specific and concrete ("never fall back to today's date"; "never use `$input.first()`"). 4.7 follows instructions literally — "don't X" is still an instruction to think about X. State what to DO.
- **Task-specific only.** Do NOT include secrets protocol or feedback protocol — CC already has both in global CLAUDE.md. Including them wastes prompt space and causes CC to lose focus on actual task content.
- **Never restate standing-protocol scope — not in a DoD, not as an end-state.** The rule above says restating protocol wastes space. Restating *scope* does worse: it manufactures a conflict CC must resolve mid-run, and the prompt is the weaker authority. Evidence: the cascade prompt's DoD asserted the end state "`Sonnet-Cascade/` empty," which is only reachable if CC also archives the build log — contradicting §5, where CC archives 3 artifacts and COO archives the build log after reading it. The DoD was unsatisfiable as written. CC correctly overrode the prompt with the standing rule. Reference the protocol ("archive per §5"), never restate what it covers, and never encode its outcome as a DoD checkbox. **Resolve at source:** if the standing scope is wrong, fix the rule — a prompt is not the place to litigate doctrine.
- **Derive counts from the enumerated source; never hand-count.** Any count a prompt asserts — files touched, rows changed, artifacts moved — is computed from the thing that enumerates them, normally the prompt's own row table. If the prompt carries a row table, the count *is* `len(table)`: state it as derived, or omit it and let the table stand. Evidence: the cascade prompt said 13 files; its row table enumerated 18. A wrong count in a DoD is a false halt condition — CC stops early or flags a phantom deviation, and the review cycle goes to reconciling a number that was never load-bearing. This generalizes the node-count gate already in `references/PROMPT-TEMPLATE.md` §COO PRE-WRITE GATE (v1.26 — "derive `{n}` as `len(nodes)` from the GET, never eyeball-scan the node list") from n8n nodes to every countable in a prompt; the gate was right, it was just scoped to one artifact type.
- **Migration gate (before writing IMPORTANT NOTES).** For each gotcha you're about to add, check: is it already in `.claude\rules\n8n-workflows-core.md`, `~/.claude/CLAUDE.md`, or a CC skill? If yes → reference it ("Standard n8n conventions apply"), don't repeat it. If no but you've seen it in 2+ prior prompts → migrate it UP first (rule/skill/CLAUDE.md), then reference it. Every prompt is a decision point: "does this belong here or one layer up?"
- **Embed quality criteria.** If there's an audit checklist, testing rubric, or acceptance criteria — put it in the prompt. CC builds to pass, not builds then fixes.
- **Reference secrets by path.** Never inline API keys. Always: "Read from `.secrets/{file}`"
- **Check anti-patterns — index first, then grep the rows. Never range-read.** Read `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then) before writing. If a similar task has failed before, embed the prevention rule. **The scan is two reads, in this order:** (1) the **`AP INDEX — DERIVED`** table — one line per active AP (`# | tags | title`), generated, complete — plus the curated TASK-TYPE INDEX above it; pick your numbers from those. (2) Fetch the chosen rows **by grep with `contextLines: 0`** — `Desktop Commander:start_search`, `searchType: content`, pattern `^\| *(N1|N2|N3) \|`. That returns your rows and nothing else. **Do NOT `read_file` with offset/length to reach a row.** AP rows are 300-800 words each and are ordered chronologically, not numerically — the 6 rows you want are scattered across a 25-row span, so a ranged read pulls ~20 rows of full-text you did not ask for. Example: a scan needing 17 rows range-read ~45, ~25 of them pure waste, with the 1-line-per-AP index sitting unused in the same file. The waste is per-prompt and permanent; the grep is one call.
- **Verify exact-text assertions via Grep.** If a prompt claims "line N contains X" or "row K is literally Y", Grep or `read_text_file` the live file BEFORE writing the assertion into the prompt. Stale exact-text assertions cause mid-execution diagnostics for CC. Paraphrase notes ("the FIX row") are fine; exact-text claims that turn out wrong are not. Evidence: plan said "SKILL.md line ~119 has `| FIX | references/task-types/FIX.md |`" — no such row existed (generic `{TYPE}.md` reference instead).
- **Verify DB schema assertions via live safe-sql.** Any prompt that names a DB table, column, constraint, enum type, or asserts a schema fact MUST be live-verified at prompt-write time via safe-sql query against `information_schema` / `pg_catalog` — not from memory, doc references, or prior-session notes. Record the verification query + result in §6 CONTEXT → Live State Verification → Schema verification. The pre-write gate row "Schema verification" in `PROMPT-TEMPLATE.md` carries the canonical query patterns; this Critical rule makes invocation mandatory rather than implied. Evidence: Opus asserted `aos_workflow_execution_log` (table didn't exist) and a `layer` CHECK constraint (actual constraint name + check_clause differed); CC discovered both at mid-dispatch INSERT failures. **Self-check before dispatching:** every DB entity name in the prompt has a matching safe-sql result in §6 Live State Verification — if any entity is asserted without a matching result, STOP and verify. "We built this in that session" is not verification; tables can be renamed, dropped, or never actually deployed.
- **Document dogfood environment constraints.** When a prompt requires CC to dogfood a new pattern (use a subagent, a new rule, a newly-installed skill), preemptively list known environment constraints that could break the dogfood. Example: "Haiku subagents cannot authenticate to Google Drive / n8n MCP — subagent-based Drive reads will fail; fall back to main-context reads and note the auth limitation in feedback." Prevents the failure being mistaken for a pattern flaw.

**Template structure (11 fields):**
```
Recommended model: Sonnet | Haiku | Opus
Recommended effort: low | medium | high | xhigh | off
Recommended budget: {time, iterations, or token cap}
/goal {acceptance_condition}           ← new: Jordan sends this as first CC message
1. CC ACCESS CHECK      (verify access before starting)
2. RESOURCES            (reads and writes — for conflict check)
3. AP SCAN              (anti-patterns checked, visible in prompt)
4. TASK                 (what to do)
5. CONTEXT QUERIES      (semantic searches before planning)
6. CONTEXT              (files, endpoints, dependencies)
7. TASK-TYPE SECTION    (BUILD/FIX/TEST/DEPLOY/etc. spec)
8. IMPORTANT NOTES      (gotchas, constraints)
9. EXECUTION POLICY     (stop criteria, fallback, verification method)
10. DEFINITION OF DONE  (checkboxes incl. feedback file)
```

### 3. HAND OFF TO JORDAN

Tell Jordan:
```
Paste into a fresh CC session: /goal {acceptance_condition from prompt header}. Read the full task at "{full filepath}" — use Plan Mode, then execute.
```

One message. `/goal` leads it — ordering supplies the anchor, not a separate send. (v1.22 collapsed the old two-step dispatch; the two-message form is retired.)

### 4. PROCESS CC RESULTS

When Jordan relays CC's results:

1. **Read the feedback file** CC wrote (same folder as prompt, `CC-FEEDBACK-{VERB}-{SUBJECT}.md`)
2. **Read the build log** if it exists (`CC-BUILD-LOG-{VERB}-{SUBJECT}.md`). Build logs capture incremental progress, errors, and decisions — especially valuable when CC hit compaction during the task and feedback quality is degraded. The build log has the pre-compaction detail.
3. **Verify outcomes** against the Definition of Done in the prompt
4. **COO VERIFICATION GATE (workflow fixes only — non-negotiable):** Query `haios_health_checks` directly via safe-sql. Do NOT accept CC's self-reported result. The DB is the oracle.
   ```
   POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}}
   { "query": "SELECT id, timestamp, state, component FROM haios_health_checks WHERE workflow_id = '{id}' ORDER BY timestamp DESC LIMIT 1" }
   ```
   Cross-check the returned row against what CC wrote in its feedback file. If they don't match, the fix is NOT done. CC's feedback file existence ≠ task success. (Root cause of that session remediation loop.)
5. **Process improvement recommendations:**

| CC recommends... | COO action |
|---|---|
| Prompt template change (universal scaffolding) | Update `references/PROMPT-TEMPLATE.md` |
| Prompt template change (task-type-specific) | Update `references/task-types/{TYPE}.md` |
| New anti-pattern | Add to `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical) |
| CLAUDE.md change (global) | Draft update, write to `~/.claude/CLAUDE.md` via Desktop Commander |
| CLAUDE.md change (project) | Draft CC prompt to update project CLAUDE.md |
| Hook change | Draft spec, CC prompt to implement |
| New skill needed (COO skill, installs to `/mnt/skills/user/`) | Flag for Jordan, use `skill-creator-skill`. After creation, add to session-context.md "Skill installs pending" (doctrine). |
| New skill needed (CC-operational, runs in CC sessions) | Flag for Jordan. Create under `~/.claude/skills/[skill]/`. Drive mirror propagates at next session-end sync (cadence). No "Skill installs pending" entry needed. 6 CC skills in this track. |
| Skill edit applied by CC (COO skill) | After COO verifies, add entry under NEXT SESSION → "Skill installs pending": `` `[skill-name]` — change summary (Sxxx). Canonical: `AOS/Skills/[skill-name]/` ``. Tell Jordan: install on Claude Desktop from canonical. |
| Skill edit applied by CC (CC-operational skill) | After COO verifies, rely on session-end sync to mirror edit to Drive mirror at `AOS/CC-Config/skills/`. No staging needed — CC is already live on the edit. See `HAIOS/Architecture/CANONICAL-GAP-DECISION.md` for track governance. |
| Tool/MCP improvement | Log in feedback log, evaluate ROI |
| `.claude/rules/` addition | Draft rule file, CC prompt to install |
| `.claude/settings.json` change | Draft update, write via Desktop Commander |

6. **Update feedback log:** Add one-line entry to `references/FEEDBACK-LOG.md`
7. **Archive and verify:** Move the prompt + feedback + feedback-processed to the flat `Archive/` (per the `archive` skill — directly in `MRMINOR/Archive`, no `CC-Prompts/` or range subfolders), then verify they're gone (see Step 5). **The build log stays until you have read it (item 2); archive it once review is complete.**

### 5. ARCHIVE + VERIFY

**CC archives 3 artifacts and leaves the build log in place. COO archives the build log after reading it.**

CC moves these (per `process-feedback` Phase 5):
```
{project}/CC-PROMPT-{VERB}-{SUBJECT}.md                →  Archive/S{session}-{verb}-{subject}-prompt.md
{project}/CC-FEEDBACK-{VERB}-{SUBJECT}.md               →  Archive/S{session}-{verb}-{subject}-feedback.md
{project}/CC-FEEDBACK-PROCESSED-{VERB}-{SUBJECT}.md     →  Archive/S{session}-{verb}-{subject}-feedback-processed.md
```

COO moves this one — **only after Step 4's review is done**:
```
{project}/CC-BUILD-LOG-{VERB}-{SUBJECT}.md              →  Archive/S{session}-{verb}-{subject}-build-log.md
```

**Why the split:** the build log is the COO's verification instrument — Step 4 item 2 requires reading it, and it carries the pre-compaction detail that a degraded feedback file lacks. Archiving it at CC-closeout puts the evidence in a cold, unindexed store in the same motion as the self-report claiming the run was fine. Authority for CC's side: `~/.claude/rules/cc-task-prompts.md` §After writing feedback — "Leave build log in place for COO review (do not archive it)", a global rule loaded every CC session.

**Verification gate (do not skip):** After CC's archive, list the prompt's source directory. **The build log is the ONLY `CC-*` file that may remain.** Any other `CC-*` file left behind = incomplete archive — CC's `process-feedback` skill sometimes claims to archive but leaves files in place. COO is the last line of defense.

```
Filesystem:list_directory → {project directory}
Expect: exactly one CC-BUILD-LOG-* file, nothing else matching CC-PROMPT-*, CC-FEEDBACK-*, CC-FEEDBACK-PROCESSED-*
```

If orphans found → move them now. Do not defer to session end.

**Denominator:** CC's archive report is `N/3`, not `N/4`. A report of `3/3` with the build log retained is CORRECT and complete — not a skipped step. Do not read it as a deviation; the build log is excluded by design, not by omission. (See the anti-pattern log — the inverse error cost a full review cycle.)

Archive is ONE flat folder (`Archive\`) — no `CC-Prompts/`, no range subfolders. If you find any nested/range archive folder, do not add to it; flag it. Archive is NOT semantically indexed (prevents search pollution). The learnings live in the skill's reference files, not the archive.

## Handoff Contract

After writing a CC prompt, state:
- Prompt file: `{full path}`
- Task type: BUILD / FIX / TEST / DEPLOY / RESEARCH / REFACTOR / MIGRATE / CREATE / AUDIT
- Jordan dispatch (one message): `/goal {acceptance_condition}. Read the full task at "{filepath}" — use Plan Mode, then execute.`
- Expected CC output: {what Jordan should see when CC finishes}
- Post-CC actions: {what COO needs to do after — verify, update docs, etc.}

After processing CC feedback, state:
- Feedback absorbed: {what was learned}
- Files updated: {skill refs, CLAUDE.md, anti-patterns, etc.}
- Archived: {prompt + feedback paths}

## Dependencies

- Desktop Commander (write prompt files, read feedback, archive)
- Relevant domain skills (workflow-build, database, etc. — for quality criteria to embed)
- `references/PROMPT-TEMPLATE.md` (router — loaded every prompt-write; template v3)
- `references/task-types/{TYPE}.md` (task-specific spec + session learnings — loaded once per prompt, based on task type). **Exception: FIX** uses two files: `FIX-CORE.md` (spec + subagent guidance, ~2KB — COO loads this when writing FIX prompts) + `FIX-LEARNINGS.md` (+ session history, COO-only reference for selecting §8 IMPORTANT NOTES; CC does NOT load it at runtime).
- `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then — checked before writing any prompt. `references/ANTI-PATTERNS.md` is a redirect stub; do not read it)
- `references/FEEDBACK-LOG.md` (updated after processing each feedback file)
- CC's global CLAUDE.md contains: Secrets Protocol, Feedback Protocol + template, archive instructions

## Authority

- Writing prompts: Tier 3 (autonomous)
- Processing feedback into skill updates: Tier 3 (autonomous)
- Processing feedback into CLAUDE.md / hook / settings changes: Tier 2 (inform after)
- Processing feedback requiring new tools or spend: Tier 1 (approval required)

## Known Anti-Patterns

See `HAIOS\Knowledge\BestPractices\ANTI-PATTERNS.md` for the full list — that path is canonical since then; `references/ANTI-PATTERNS.md` is a redirect stub. Start from its TASK-TYPE INDEX. Top failures:

| Anti-Pattern | Rule |
|---|---|
| Feedback protocol skipped on long prompts (#21) | Feedback protocol lives in CLAUDE.md now, not prompts. DoD references it. |
| Prompt assumes COO session context (#3) | Prompts must be 100% self-contained |
| Migration rename assumed internal naming (#20) | Specify naming source: n8n, AOS convention, domain registry |
| Regex specs without literal test strings (#19) | Include exact input→output test table for regex-heavy specs |

## Changelog

| Version | Changes |
|---|---|
| 1.38 | **`PROMPT-TEMPLATE.md` §Known Caveats — the DDL caveat inverted from an assumption to a probe.** It read *"Supabase port 5432 is unreachable from Jordan's machine — psycopg2 DDL will fail at connection"* (Confirmed 2026-05-04) and steered every DDL prompt straight to Jordan-manual SQL Editor. **`BUILD-AGENCY-SUPPLY-TABLE` made the probe its first §1 ACCESS CHECK step and psycopg2 connected on the first try** — then ran a 6-step migration (ALTER TYPE + 5 enums + CREATE TABLE + 6 indexes + trigger + RLS), idempotent on re-run, plus 3 write-assertion smoke tests with rollback, autonomous, zero Jordan touches. Replaced with **probe-and-branch**: connects → PATH A autonomous; fails → PATH B Jordan SQL Editor; record which fired. **The corpus had held both claims at once for many sessions** — said psycopg2 is not viable while *requires* it for write-assertion tests and pre-approved `migrate-*.py` psycopg2 scripts in `settings.json` *because they run*. Nobody collided them because the caveat said don't, so no prompt tried, so it was never contradicted — **a "don't" nobody tests is never contradicted**, which is that session's shape (a "cannot" is usually an untested "haven't") sitting inside a gate rather than a session. Companion edit: that session's rule cell carries the amendment retiring its clause (3); the rest of that session stands (SQL Editor is the reliable fallback; safe-db-write blocks TRUNCATE; REST DELETE times out >~50K rows). **Second fix, same block — a miscitation caught by KEY LAW 4:** the stop-hook paragraph cited an anti-pattern number; that entry exists and is a graduated entry about `export KEY="value"` leaking to shell history, unrelated. The paragraph's content is verbatim the anti-pattern on the DDL stop-hook loop — both its option (a) and option (b) match that row's rule text. Repointed. The number was checked before the claim was made; an absence would have been a claim about the grep. Also generalized the caveat's scope header FIX/MIGRATE → FIX/MIGRATE/**BUILD** — was a BUILD and the caveat did not name it. |
| 1.37 | **§2 Critical rules — "Check anti-patterns" now specifies the read method, not just the file.** The rule named the target and left retrieval to improvisation; every prompt-write paid for that. Scan is now: read the generated `AP INDEX — DERIVED` table (1 line per AP) + the curated TASK-TYPE INDEX, pick numbers, then grep the chosen rows with `contextLines: 0`. **Range-reading to reach a row is now explicitly out.** Origin: LNI-PLACES-JOIN prompt-write — COO index-scanned correctly and grepped correctly for line numbers, then fetched the 17 rows via four `read_file` offset/length calls, pulling ~45 rows to get 17. AP rows are ordered **chronologically, not numerically**, so any contiguous span containing your targets also contains everything appended between them; at 300-800 words per row the neighbors cost more than the scan. The 1-line index that makes this unnecessary was in the same file, three rows into a `head 85` that stopped short of it. **Caught by Jordan mid-session, not by the skill** — the gate said *what* to read and was silent on *how*, and silence defaults to the expensive method. No prompt content was wrong; this is pure retrieval waste, which is why it survived many sessions unnoticed. |
| 1.36 | **Storm-routing citations repointed (ANTI-PATTERNS renumber, KEY LAW 1: keys are bare integers).** 2 live citations updated: §1 Storm-routing gate header, changelog row 1.27 (origin entry — numbers updated in place since this whole file is in-scope for the cascade, unlike standalone historical-audit records). Cascade: `ap93-renumber-cascade-completion.md` TASK A. |
| 1.35 | **`references/task-types/AUDIT.md` — new §Citation/frequency-scan audit notes (4 bullets), from the AP-citation-scan process-feedback (CC-drafted, COO-vetted).** Headline bullet: **never use a "known fact" as BOTH prompt context AND a pass/fail DoD control.** The prompt asserted "#108 has zero rows" as trusted-given context, then made "Table D includes #108" a literal DoD checkbox. Live verification found #108 *does* have a row — so the control failed for a legitimate reason (stale premise), and the report had to spend a section explaining that a red X was a green check. A control built on a premise cannot separate *"the scan is broken"* from *"the premise was wrong"*; phrase such controls conditionally, or derive them from live state at execution. Remaining bullets point at the three APs logged: **#258** (a scan's negative result is an assertion about the scanner — verify absence per-instance before acting or recording), **#259** (never per-file-loop a bulk grep over `the shared drive` — 710 files hit the 3-min timeout with zero output; one `grep -r` did it in under a minute), **#260** (anchor short all-caps citation greps with `\b` — `AP[ -]#?\d` false-matched `GAP-1`/`CAP 500`/`ROADMAP 4.20`, 89% of raw variant hits). **Note the AP numbers moved:** CC drafted these as #256/#257; #256–#258 were taken at that time by the ANTI-PATTERNS key normalization (`93a`/`93b`→#256/#257) and the scanner AP (#258), so they landed at **#259/#260** — per Rule 13, numbers were derived from a live max grep, never inherited from the CC self-report. This entry records provenance + version bump; behavioral change lives in `AUDIT.md` (read live at prompt-write). |
| 1.34 | **Dispatch self-contradiction struck + 2 Critical rules added (diagnosis, deferred on context-depth).** (1) **One-message dispatch is now stated once.** The skill said "one message" in §Quick Reference L20 and §Handoff Contract L164, and "Two messages, not one" in §3 L92 — with a stale v1.20 two-step block at L24–25 sitting directly under the L20 one-message header, contradicting it inside the same section. v1.22 collapsed the dispatch to one message and `claude-instructions.md` has carried the one-message form since; SSOT wins, the v1.20 remnants are struck. What survives is the *reason* the two-step existed: `/goal` must precede the file read. One message preserves that by ordering. **The bug was v1.22's:** it updated the three places that named a message count and missed the block that merely *showed* one — a rewrite that greps for the claim and not for the illustration leaves the illustration behind, and the illustration is what a reader copies. It stayed live for many sessions. (2) **§2 Critical rules — "Never restate standing-protocol scope"**: the cascade DoD asserted end-state "`Sonnet-Cascade/` empty," reachable only if CC archived the build log too, contradicting §5 (CC archives 3, COO archives the build log post-review); DoD unsatisfiable as written; CC correctly overrode the prompt with the standing rule. Prompts reference protocol, never restate its scope or encode its outcome as a checkbox — resolve at source. (3) **§2 Critical rules — "Derive counts from the enumerated source"**: the prompt said 13 files, its own row table enumerated 18. Generalizes the v1.26 node-count gate (`len(nodes)`, never eyeball-scan) from n8n nodes to every countable in a prompt — the gate was correct, merely scoped to one artifact type. CC recommendation, adopted. |
| 1.33 | **Build-log archive conflict resolved — §5 + Step 4 item 7 now match the CC-side rule.** Three live protocols disagreed on whether CC archives the build log: `~/.claude/rules/cc-task-prompts.md` ("Leave build log in place for COO review") vs `process-feedback` Phase 5 ("ALL 4 artifact files must be moved... missing build-log is a failure", report `N/4`) vs this skill's §5 (build-log in the move set + "confirm NO `CC-*` files remain"). CC cannot satisfy both; every run had to violate one. **cc-task-prompts wins on merit** — the build log is the COO's verification instrument (Step 4 item 2 mandates reading it; it holds the pre-compaction detail a degraded feedback file lacks), so cold-storing it at CC-closeout destroys the review it exists for. §5 rewritten: CC archives 3, COO archives the build log post-review; the orphan gate now expects exactly one `CC-*` survivor; explicit `N/3` denominator note added. Companion edit: `process-feedback` Phase 5 (canonical `~/.claude`) retargeted to `N/3` + DO-NOT-ARCHIVE row. **Cost of the conflict:** it presented as agent fabrication and was mis-recorded as a fabrication, which blamed CC for citing a "nonexistent protocol" that in fact exists and loads every session — so the real collision survived the AP and re-fired at that time unchanged. rewritten same session. |
| 1.32 | **Remaining `references/ANTI-PATTERNS.md` stub refs retargeted to canonical (6 total).** SKILL.md §2 Critical rules "Check anti-patterns", §4 action table "New anti-pattern" row, §Dependencies; `references/PROMPT-TEMPLATE.md` §COO PRE-WRITE GATE anti-patterns row, §3 AP SCAN, §8. All now point at `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then); the `references/` copy has been a redirect stub for many sessions and every doc still routed readers through it. Completes the fix-on-sight item that v1.31 only half-closed. Version bumped separately because v1.31 shipped and was installed before these edits — a content change without a bump is the same drift class this session deleted elsewhere. |
| 1.31 | **AP entry counts deleted, not updated; canonical ANTI-PATTERNS path corrected.** §Known Anti-Patterns cited "85 active entries as of then" and pointed at `references/ANTI-PATTERNS.md` — a redirect stub since then. Both fixed in one line. The count was **removed rather than refreshed**: APs are consumed by number via the TASK-TYPE INDEX, never by count, so no reader existed — a tree-wide grep for `Active entries` found only prose, and two of those mentions (`ANTI-PATTERNS.md` AP-NUMBERING, `process-feedback/SKILL.md`) were instructions to *distrust* the count. A field with no consumer, a per-addition edit cost, and a demonstrated drift record is a liability; the compute command was always the only load-bearing part. Companion edits: `ANTI-PATTERNS.md` (dropped `Active entries:` line + cached `max=229`, stale by 20 vs live max=249) and `process-feedback/SKILL.md` (dropped its now-orphaned distrust clause). Trigger: COO surfaced the drift as a cascade item; Jordan asked what the number was *for*; nothing. |
| 1.30 | **Reference hygiene — monitoring-registry + schedule-trigger verification.** Three `references/task-types/` edits: (1) `BUILD.md` liveness cadence table — `Daily` row corrected from `NULL (inherits 24h default)` to explicit `24` (a present NULL row *suppresses* monitoring; NULL≈default was operationally wrong); (2) `BUILD.md` `On-demand` row de-legacied — points to registry-NULL suppression + the observing-caller taxonomy, drops the instruction to grow the frozen `PG: Find Silent Failures` NOT-IN list; (3) new `FIX-CORE.md` § Schedule-Triggered Verification + a `BUILD.md` pointer — FIX/BUILD prompts on schedule-triggered n8n workflows must plan the Jordan-fired manual-run verification path (n8n MCP auth unreliable in CC; `POST /run` → 405). Taxonomy SSOT = `HEALTH-CHECK-PATTERNS.md` §On-Demand (folded). Records provenance + version bump. |
| 1.29 | **BUILD.md Session Learnings — verbatim-SQL rule added.** BUILD/FIX prompts that add or modify a concatenated SQL INSERT/UPDATE in a Code node must supply the exact column-list + VALUES template verbatim (not "map field X into the INSERT") and require a DoD quote-balance unit test. Origin: the Intel Step-3 tag build described the INSERT change rather than specifying it; CC's hand-built `Parse Response` grouped insertSql dropped the closing quote on `signal_source` → best_practice-path syntax error, caught only by COO live-code review. Change in `references/task-types/BUILD.md`; this entry records provenance + version bump. |
