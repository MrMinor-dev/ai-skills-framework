---
name: process-feedback
version: 1.9
description: "Process a CC feedback file and draft proposed environment updates (anti-patterns, CLAUDE.md, rules, skills). Level 2: proposes only, never auto-applies Tier 1 changes. Note: CLAUDE.md is Tier 1 (never auto-applied) as of then."
triggers:
  - process feedback
  - absorb feedback from
---

# Process Feedback

Reads a CC feedback file → classifies changes by tier → auto-applies Tier 2/3 → drafts Tier 1 for COO.

**Level 3:** CC applies Tier 2/3 autonomously, drafts Tier 1 for approval.

---

## Phase 1: READ FEEDBACK

Read the specified file. Extract from these sections (skip gracefully if absent — sparse feedback is normal):

- **Outcome** — SUCCESS / PARTIAL / BLOCKED
- **Definition of Done** — blocked/incomplete items
- **Prompt Quality** — "What was ambiguous", template recommendations
- **Environment Recommendations** — table (Layer | Rating | Note) or freeform subsections
- **Task-Specific Notes** — actionable bullet points

Capture: session number (S{N}), task name, date.

---

## Phase 2: CLASSIFY + PRIORITIZE

See `references/classification.md` for the full signal→target→tier table.

**Tier meanings:** 3 = autonomous after COO review | 2 = apply + notify | 1 = draft only

**Numbering (required before any new anti-pattern):** Compute the next AP number - do NOT read the full file. New entry number = (max existing AP number) + 1. Get the max with one command (entries are appended chronologically with gaps, so the LAST row is NOT the max, and a partial read produces a false max):

```
grep -oE '^\| *[0-9]+' "HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md" | grep -oE '[0-9]+' | sort -n | tail -1
```

This scans active + GRADUATED rows and returns one integer - no need to load the doc into context. Always run it; the master file maintains no entry count to read instead (removed - it had no consumer and drifted). (See the `AP-NUMBERING` comment at the top of the master file.) Assign a `[domain]` tag per the tag definitions in the master file header - prepend to the Pattern column value. Tags: `[n8n]`, `[db]`, `[cc-ops]`, `[infra]`.

**Deduplication (required before any new anti-pattern):** Check whether the pattern already exists before adding. Instead of reading the whole file, grep ANTI-PATTERNS.md for the candidate's `[domain]` tag plus 2-3 distinctive keywords from the failure, and read only the matching rows. If a similar entry exists, propose an update to that entry instead of adding a new one.

**Cross-reference search (required before adding any new AP):** Before classifying a feedback item as a new AP candidate, also search these reference files for existing coverage of the same pattern:
- `AOS/Domains/Infra/N8N-API-REFERENCE.md` — pitfalls section
- `AOS/Skills/cc-prompt-skill/references/task-types/FIX-LEARNINGS.md` — session learnings
- `HAIOS/Knowledge/BestPractices/CC-DEVELOPMENT-PLAYBOOK.md` — broader patterns

If the pattern already exists in any of these files, do NOT add a new AP — instead note the existing canonical location and propose a CLAUDE.md sub-bullet or cross-ref if the gap is merely discoverability. Root cause: added a duplicate AP about safe-sql 0-row results already documented as N8N-API-REFERENCE.md Pitfall #27.

---

## Phase 3: DRAFT CHANGES

Produce exact, copy-paste-ready content for each item.

**Anti-pattern (new):** `| {max+1} | {short name} | {what happened} | S{N} | {prevention rule} |`

**Append mechanics (mandatory - the row format above is NOT sufficient on its own).** ANTI-PATTERNS.md does not reliably end with a trailing newline. Appending the row directly concatenates it onto the last existing row, producing a welded `... |{new row}` line: the new AP is swallowed into the previous AP's final cell, corrupting BOTH rows. Before appending any row: read the file's final character. If it is not a newline, write one first, then the row, then a trailing newline. Safe pattern:

```python
with open(path, 'r', encoding='utf-8') as f: content = f.read()
if not content.endswith('\n'): content += '\n'
content += new_row.rstrip('\n') + '\n'
with open(path, 'w', encoding='utf-8') as f: f.write(content)
```

Never append a table row with a bare append-mode write that assumes a trailing newline exists. (root cause: welded onto that session's rule cell AND written a second time; both rows corrupted. is the CC-closeout-fabrication entry - the run that logged an AP broke the AP file.)

**AP-index regeneration (mandatory — COO-approved, D-13 half 1).** ANTI-PATTERNS.md carries a generated index in a sentinel-delimited region (`AP-INDEX:BEGIN` / `AP-INDEX:END`). It goes stale the instant a row is appended or updated, and a stale index makes the new AP unreachable to every reader that uses the index instead of scanning the file. After ANY append or update to ANTI-PATTERNS.md, run:

```
py "HAIOS\Tools\ap-index\generate_ap_index.py" --write
```

Then verify: `--check` exits clean, and the last row inside the sentinel region matches the highest AP number in the file. Do not skip this because the append "was only one row" — one row is exactly the failure. (evidence: were appended without regeneration; the index sat at #261 and coverage went 100% -> 98.6% within a single session. Measured decay rate is one session.)

This is a local script — no MCP or Drive round-trip. It fires here because this is the append path CC owns; other append paths are covered by a separate session-end staleness check.

**Anti-pattern (update):** `Existing #{N}: {current} → Proposed: {updated}`

**Feedback log entry:** `| {YYYY-MM-DD} | {session} | {task} | {key learning} | {absorbed into} |`

**CLAUDE.md:** `Section: {name}` / `Add after: "{line}"` / `Content: {exact text}`

**Rule file:** Full file content including frontmatter.

**N8N-API-REFERENCE:** `Section: {name}` / `Add: {content}`

---

## Phase 4: APPLY + OUTPUT

**Tier 3 (Autonomous):** Apply changes directly to target files. **Verify each write landed before logging as Applied** (see verification gate below). Log what was changed.
**Tier 2 (Inform After):** Apply changes directly to target files. **Verify each write landed before logging as Applied.** Log what was changed. Note for COO.

**AP write verification gate (mandatory for any ANTI-PATTERNS.md addition or update):**
After writing, run BOTH counts against the canonical file:

```
grep -c '^| {N} |' "HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md"   # anchored   -> must be 1
grep -c '| {N} |'  "HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md"   # unanchored -> must be 1
```

Both must return exactly **1**. Read the truth table before reporting:

| anchored | unanchored | Meaning | Status |
|---|---|---|---|
| 1 | 1 | clean single row at line start | **Applied** |
| 0 | 0 | write did not land | `FAILED - write did not land` |
| 0 | 1+ | **row welded mid-line onto the previous row** (missing newline) | `FAILED - row concatenated, both rows corrupted. Repair before reporting.` |
| 2+ | 2+ | duplicate row written | `FAILED - duplicate. Remove extras before reporting.` |

Only on `1 / 1`: run `grep -n '^| {N} |'` and embed the matched line verbatim in the Applied table `Status` cell: `Applied - grep match: | {N} | {short name} | ...`

**A FOUND/NOT-FOUND check is NOT sufficient and is retired.** An unanchored existence grep returns a hit for a duplicate AND for a welded mid-line row - it passes on both real failure modes. Count, anchor, compare. (Note: CC reported the write "grep-verified"; the grep was truthful and the file was corrupted two different ways. remains why the gate exists at all: CC reported Applied on the entry never existed.)

A pass/fail claim without the literal grep match is not COO-verifiable. Never report Applied without embedding the actual grep output.
**Tier 1 (Approval Required):** Draft only. Do NOT apply. Include exact copy-paste-ready content.

**FEEDBACK-LOG cap + write-verification (enforce before writing report):**
The FEEDBACK-LOG canonical SSOT is `AOS/Skills/cc-prompt-skill/references/FEEDBACK-LOG.md`. ALWAYS write the FULL path — NEVER a bare `FEEDBACK-LOG.md`. A bare name can resolve to a stale duplicate in the working directory (e.g. a sibling of ANTI-PATTERNS.md) and the write lands silently in the wrong file. (root cause — phantom retired.)
1. Append the new entry to the canonical path above.
2. **Landed-in-canonical gate (mandatory — mirrors the AP-write gate):** after appending, grep the CANONICAL path for the new row's `S{N}` token and embed the matched line verbatim in the report `Status` cell: `Applied — grep match (canonical): | {date} | S{N} |...`. If the grep returns empty, mark `FAILED — write did not land in canonical FEEDBACK-LOG`. Do NOT claim Applied. A row-count check ("10 rows") is NOT verification — it is path-blind and passes against ANY same-named file (root cause).
3. Cap: after the VERIFIED append, count rows in the canonical table. If count > 10, delete the oldest row(s) until count = 10. Rolling 10-entry window — old entries are absorbed into ANTI-PATTERNS.md and task-type files; no separate archival.
Entry format: 1 line, ≤200 chars (Date | Session | Task | Outcome + Key Metric | Absorbed Into) — keep it tight; this file is read on every process-feedback run and adds compaction pressure when verbose.

After applying, write report to: `{same directory as input}/CC-FEEDBACK-PROCESSED-{SUBJECT}.md`

```
# Feedback Processing Report
**Input:** / **Session:** / **Processed:**

## Applied — Tier 3
| # | Change | Target | Status |
## Applied — Tier 2 (COO: review these)
| # | Change | Target | Status |
## Draft Only — Tier 1 (COO approval needed)
{exact content to apply}
## Already Absorbed

## Summary
- {N} applied (Tier 3) / {N} applied+notified (Tier 2) / {N} drafted (Tier 1) / {N} already absorbed
```

---

## Phase 5: ARCHIVE ARTIFACTS

**One archive, flat, cold.** The canonical contract lives in the `archive` skill (`~/.claude/skills/archive/SKILL.md`; mirror `AOS/CC-Config/skills/archive/SKILL.md`) — this phase implements it for CC artifacts. Target is `Archive\` — ONE folder, flat, files directly inside it. **NO `CC-Prompts/` folder, NO session-range subfolders (`\` etc.), NO `misc\`, NO nested Archive.** If you encounter a range subfolder or a nested Archive, do NOT add to it — flag it for COO. (decision: range subfolders + per-layer archives were retired; the flat store + self-locating filename replace them.)

**CRITICAL: The build log STAYS IN PLACE for COO review — do not archive it.** The other 3 artifact files must all be moved. A partial archive (prompt only, or missing feedback/feedback-processed) is a failure — do not mark this phase complete until all 3 are confirmed moved.

**Why the build log is the exception:** COO reads it to verify the run — that review is the whole reason it exists, and it cannot happen if the log is cold-stored in the same breath as the report claiming the run went fine. Authority: `~/.claude/rules/cc-task-prompts.md` §After writing feedback ("Leave build log in place for COO review (do not archive it)") — a global rule loaded every session. COO archives it after review.

**Rename and move each artifact** to the flat archive (VERB-SUBJECT derived from the feedback filename, lowercased). In a flat store the filename is the sole carrier of provenance, so the self-locating `S{N}-{verb}-{subject}-{type}.md` name is mandatory:

| Source filename | Archive path (flat) | Required? |
|---|---|---|
| `CC-PROMPT-{VERB}-{SUBJECT}.md` | `Archive\S{N}-{verb}-{subject}-prompt.md` | Required |
| `CC-FEEDBACK-{VERB}-{SUBJECT}.md` | `Archive\S{N}-{verb}-{subject}-feedback.md` | Required |
| `CC-FEEDBACK-PROCESSED-{VERB}-{SUBJECT}.md` | `Archive\S{N}-{verb}-{subject}-feedback-processed.md` | Required (written in Phase 4) |
| `CC-BUILD-LOG-{VERB}-{SUBJECT}.md` | **DO NOT ARCHIVE — leave in source folder for COO** | Never (COO archives post-review) |

**Steps:**
1. For each of the 3 archived artifact types above: check if the file exists in the source directory. If it exists, move it to its flat archive path (directly in `Archive\` — never into a subfolder) and verify the move. **Leave `CC-BUILD-LOG-*` where it is.**
2. If the destination filename already exists in `Archive\` (name collision), append a uniqueness suffix before the extension (`...-prompt__dup1.md`) — never overwrite.
3. List the source directory and confirm the only `CC-*` file remaining is the build log. Log this check in output. Any other `CC-*` file left behind = incomplete archive.
4. Report the archive result explicitly: `"Archived N/3 artifacts (flat → Archive\): [list moved files]. Build log retained in source for COO review: [path]."` — never say "archived" without specifying which files and confirming the flat target.

**Archive completeness rule:** If the prompt was moved but feedback or feedback-processed were not (because they were in a different location or not yet written), flag this explicitly in the report: `"PARTIAL ARCHIVE — {N}/3 moved. Remaining: {list}. COO must complete."` Do not silently leave files behind. The retained build log is never a partial archive.

---

## Safety Rules

- **NEVER auto-apply Tier 1** (rules, hooks, skills) — draft + await approval only.
- **Tier 2/3 auto-apply is expected.** If a file write fails, log the failure and continue — do not block the report.
- **NEVER modify this skill** without explicit COO approval.
- **Flag conflicts:** recommendation vs. existing anti-pattern/rule → note conflict, do not resolve.
- **Deduplication:** check ANTI-PATTERNS.md before adding any new entry.
- **Offer after every task:** *"I wrote a feedback file. Want me to process it now?"*

---

## Changelog

| Version | Changes |
|---|---|
| 1.9 | **Phase 3: AP-index regeneration is now mandatory after any ANTI-PATTERNS.md append or update.** `generate_ap_index.py --write`, then verify `--check` clean + sentinel-region max key == file max key. Rationale: R-1 made the index derived and 100%-covering (206 APs; 127 previously write-only), and it went stale in ONE session — CC appended during the same session's site build without regenerating, dropping coverage to 98.6% and making 3 new APs unreachable to any index-based reader. Measured decay rate = 1 session, so no scheduled or session-end-only mechanism is sufficient on this path. Coupled to the append because that is the only event that introduces staleness. This is D-13 half 1 of 2 (`HAIOS/Architecture/GOVERNANCE-SUBSTRATE-AUDIT-FABLE.md` §5); half 2 is a read-only staleness check at COO session-end covering the append paths CC does not own (direct COO edit, Sonnet cascade) — COO cannot execute the generator (no `the shared drive` from container bash, no execute on Filesystem MCP), so detection and regeneration are split by actor capability. Self-modify gate satisfied: explicit COO approval. |
| 1.8 | **Table-row append hardened - newline mechanics + counting write-gate.** Two live bugs, both fired in one run. (1) **Phase 3:** the skill specified the AP row *format* but never the *append mechanics*; ANTI-PATTERNS.md has no reliable trailing newline, so the append welded into that session's final cell (`... checked. |  | 250 | [infra]...`), corrupting both rows - and is the CC-closeout-fabrication entry, i.e. the run that logged an AP broke the AP file. Added mandatory read-last-char / pad-newline / append pattern with a copy-ready Python snippet; bare append-mode writes forbidden. (2) **Phase 4:** the write gate was an *unanchored existence* grep reporting FOUND/NOT FOUND - structurally blind to both real failure modes, since an unanchored hit is returned for a duplicate row AND for a welded mid-line row. The same run wrote twice and reported it "grep-verified"; the grep was truthful. Replaced with dual anchored+unanchored `grep -c`, both must equal 1, plus a 4-row truth table naming the welded-row case (`0 / 1+`) explicitly. COO repaired both corrupted rows. Phase 2 also drops the retired "do not trust `Active entries:`" clause (that count was deleted - it had no consumer). |
| 1.7 | **Phase 4 FEEDBACK-LOG write hardened — full canonical path + landed-in-canonical grep gate (mirrors the AP-write gate).** Cap step previously referenced `FEEDBACK-LOG.md` by BARE filename with only a path-blind row-count check; a stale duplicate at `HAIOS/Knowledge/BestPractices/FEEDBACK-LOG.md` silently absorbed 2 writes (row-count passed against the wrong file). Now: full canonical path mandatory + post-append grep-match-in-canonical gate with embedded matched line, FAILED on empty. Phantom retired to Archive same session (R-A). Root cause: `HAIOS/Knowledge/FEEDBACK-LOG-MISS-ROOT-CAUSE.md`. |
| 1.6 | **Phase 5 rewritten to flat cold archive (decision).** Artifacts now move to `Archive\` flat — removed the entire session-range subfolder map (`Archive\CC-Prompts\S{range}\` + `misc\`). Self-locating `S{N}-{verb}-{subject}-{type}.md` rename retained; collision-suffix rule added; phase now defers to the `archive` skill contract. Trigger: FIX-DOCGOV-RETIRE-CHECK2 artifacts archived to the OLD range path (`Archive\CC-Prompts\\`) immediately after the migration flattened everything — the new `archive` skill was inert because this Phase 5 still encoded the retired structure. COO moved those 4 files flat manually; this edit stops recurrence. |
| 1.5 | AP write-target in classification.md repointed to canonical `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (was stub path `AOS/Skills/cc-prompt-skill/references/ANTI-PATTERNS.md`). CONFIG-PLACEMENT-DOCTRINE §6 follow-up #1. running cascade Item 4. |
| 1.4 | **Phase 2: replaced full-doc read with grep for AP numbering + keyword-grep for dedup.** CC previously read the entire ANTI-PATTERNS.md every run to find the max AP number (and to scan for dups); now computes max via a `grep... sort -n | tail -1` one-liner (one integer, no full read) and dedups by grepping domain tag + keywords. Paired with new `AP-NUMBERING` comment at top of master file. Trigger: Jordan - CC was reading the entire doc just to find the max number. |
| 1.3 | CLAUDE.md demoted Tier 2 → Tier 1 in classification.md (OPEN-6) — CLAUDE.md is every-message tax, draft-only, never auto-applied. |
| 1.2 | Added archive range rows to Phase 5 — previously all mapped to misc\. Applied canonical ANTI-PATTERNS.md path in Phase 2. |
| 1.1 | Expanded prior-coverage search scope in Phase 2 to include N8N-API-REFERENCE.md, FIX-LEARNINGS.md, CC-DEVELOPMENT-PLAYBOOK.md. Trigger: Pitfall #27 duplicate discovery (process-feedback searched only ANTI-PATTERNS.md; pattern was already canonical in N8N-API-REFERENCE.md Pitfall #27). |
