# REFACTOR Task Type

Restructure, optimize, clean up.

**Recommended effort:** `medium` (default; override to `high` for cross-module or ambiguous-behavior-preservation)

- Override to `high` for cross-module refactors or ambiguous behavior preservation
- Override to `xhigh` for genuinely novel restructuring judgment
- Override to `low` for single-file rename/extract with clear boundaries

## COO → CC Routing Check (doctrine)

**Before authoring a REFACTOR prompt for CC, COO checks the routing table below. If the answer is "COO," do not write a CC prompt — COO executes directly.**

| Target file characteristics | Route |
|---|---|
| Part of CC's permanent context tax (`~/.claude/CLAUDE.md`, `~/.claude/rules/*`, `~/.claude/settings.json`) | **COO** |
| >15 KB AND requires whole-file context (classification, restructure, large cross-section move) | **COO** |
| Surgical edit (Edit tool with small `old_str`/`new_str` match on a large file) | CC (fine) |
| New file creation | CC (fine) |
| Small file (<10 KB) of any edit type | CC (fine) |

**Why:** See `HAIOS/Architecture/C1-POST-MORTEM.md` and "CC Self-Refactor Storm." The target file must be loaded to modify it. When that file is part of CC's permanent tax OR >15 KB with whole-file judgment required, loading it combines with system prompt + skills + permanent tax + thinking burn to cross auto-compact threshold. Compaction destroys in-context judgment state (per-bullet classification). CC recovers via compact summary but can't restore judgment state. Compact summary generation itself is expensive. Feedback loop → storm. No amount of prompt engineering prevents this structurally. Route around it.

<!-- COO uses this file when writing a CC prompt for a REFACTOR task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## CURRENT STATE
{What exists now. File paths, structure, issues.}

## TARGET STATE
{What it should look like after. Constraints on what can/can't change.}

## VERIFICATION
{How to confirm refactor didn't break anything.}
```

## Classification Table Persistence (judgment-dense refactors) —

**When a REFACTOR task requires per-unit judgment (classifying bullets, categorizing sections, deciding which items move where), the spec MUST require CC to persist the classification table to the build log BEFORE running mechanical operations.**

Format to require in the prompt's §7 TASK-TYPE section:

```markdown
## CLASSIFICATION PERSISTENCE

Before writing any output file, complete classification in a build-log table:

```
| # | Source unit (bullet / section / line range) | Classification | Target |
|---|---|---|---|
| 1 | "**Connections format:**" bullet L42-46 | HYBRID | CORE (rule-only) + LEARNINGS (full) |
| 2 | Plan Phase Gates H2 L88-96 | VERBATIM-BOTH | CORE + LEARNINGS |
| ... | ... | ... | ... |
```

Append to `CC-BUILD-LOG-{verb}-{subject}.md` section `## Classification` BEFORE any file writes.
Post-compaction recovery reads this table instead of re-classifying in context.
```

**Why:** Build log captures step headings for procedural tasks but not judgment state. On that session C1, CC's build log showed "Classification complete per plan. ~25 hybrid bullets identified" without recording WHICH bullets went where. When compaction fired mid-task, CC had to re-classify, burning the budget compaction was supposed to free. The classification table on disk breaks the re-read-then-re-decide loop.

## Subagent Delegation (mandatory for tasks with 5+ file reads or 10+ EXECUTE rows)

You are an orchestrator. Do not read all target files in main context.

**File reading phase:**
Spawn Explorer subagents (Haiku) in batches of 5-6 files:
"Read files [list]. For each: current state, deviation from routing table spec.
Return 300-line max summary."
Main session receives summaries only — never raw file content.

**Edit execution phase:**
Spawn Batch Worker subagents per EXECUTE group (A, B, C...):
"Execute EXECUTE rows [A1-A5]. For each: read file, make specified edit,
re-read to verify. Return: row → edit made → verified (Y/N)."

**Compaction trigger (between groups):**
After each subagent group completes: /compact
"Keep: routing table, completed rows, current group. Drop: file contents."

## Session Learnings

**Safety gate for directory deletion:** When specifying a "verify empty before delete" safety gate, write: "MUST be empty (no files AND no subdirectories)." Writing "no files" alone is ambiguous -- an empty subdirectory passes the file check but fails `[System.IO.Directory]::Delete`. CC will stop and ask. If the intent is "no meaningful content," say so explicitly.

**Routing table notes for doc-restructure REFACTOR tasks:**
- **Verify file paths before writing prompt:** Every path in a routing table must be glob-verified against the live file system before the prompt is written. Do NOT copy paths from memory or prior docs -- files move (e.g., `HAIOS/Design/CALENDAR-ARCHITECTURE.md` vs actual `AGENCY/CALENDAR-ARCHITECTURE.md`). A single mismatched path forces CC to search mid-execution, consuming plan time.
- **Phase labels -- clarify "Now" vs "Done":** If a source doc has a phase labeled "Now" (or similar present-tense marker), state explicitly whether CC should treat those items as done (preserve as historical record) or active/pending (eligible for removal). Without this, CC must infer from context -- it may guess wrong. Recommended phrasing: "Phase A 'Now' items = current-state documentation, preserve as-is."
- **Source cleanup table must use exact heading text:** When specifying what to remove from a source doc, use the exact heading text as it appears in the live file -- not a COO-description or paraphrase. Section names drift between the time a scour report is written and when the REFACTOR prompt is authored. Mismatched heading names force CC to match by content rather than by anchor, increasing risk of wrong-section removal. Example failure: prompt said "Pending Tasks / L4 Build Requirements"; live doc had "Dependency Map (Build Order)" and "Open Questions." CC matched correctly on content, but the discrepancy required extra verification.
- **Quote exact current text for find-replace style edits:** When a REFACTOR edit replaces a section's body (rather than removing/adding), quote the exact current text in the spec -- not just a description of it. This eliminates search ambiguity and removes the verification read CC would otherwise do before editing. See Edit 8 (CC Playbook Future Enhancements) as the correct pattern: quoting the exact current pointer text made the replacement unambiguous.
- **Add a "current state" column to target files tables:** When listing target files and their edits, add a short "Current State" column: e.g., "section already a clean stub", "section does not exist -- create it", "2 items present -- remove item B". This saves the plan-time file read CC does to verify current state before editing.: 3 of 9 edits required verification reads that a current-state column would have eliminated.
- **Line numbers in routing tables are fragile across sessions:** When a routing table specifies a location by line number (e.g., "lines 209-223"), those numbers may be stale if the target file was edited between the audit session and the execution session. Use section header anchors as location references instead (e.g., `### The Insight ` rather than `lines 209-223`). CC can usually recover from stale line numbers using the action description, but this requires an extra verification read. Best practice: quote the section heading text verbatim as the location anchor.
- **Domain refresh items -- quote exact purpose text:** When a routing table item says "update purpose to match [source doc]", quote the exact purpose text verbatim in the spec. Do NOT rely on CC to read the source doc and select the right line.: Intelligence domain purpose not specified; CC read INTELLIGENCE-DOMAIN.md and chose correctly, but this is low-risk guessing -- the right call may not always be obvious.
- **Decision note items -- specify format:** When routing a "add a decision note" item to a target file, specify the exact format: blockquote (`>...`), bullet list item, or inline paragraph. The section type (reference paragraph vs. list) is not always obvious from the spec.: L4 decision notes format unspecified; CC used blockquote because section was a reference paragraph -- correct, but format should not be a judgment call for CC.
- **Promote-up routing -- specify which items:** When routing says "Promote N items to [target]", list the exact items by name or quote their headings. Without this, CC must infer which N items qualify (e.g., cross-workflow vs. workflow-specific) -- a judgment call that may guess wrong.+: "Promote 4 RUNTIME DISCOVERIES items" unspecified; CC correctly identified the 4 cross-workflow n8n patterns, but this was inference, not execution.
- **Integer-style version edge case (post-edit protocol):** Some READMEs use integer-style versions (V1, V2, V4) tied to the n8n workflow name -- these CANNOT be patch-bumped (e.g., V4.1 is not valid). The post-edit protocol "bump patch version" does not apply. For these files, update only the `updated` date. Identify integer-style versions by format (`VN` with no decimal) and note explicitly in routing table: "integer-style version, date-only bump".+: AOS-Finance-Expense-Entry V4 -- correctly date-bumped only, but the edge case was not covered in the prompt.

**Domain doc steps -- verify table/section heading name before referencing:** When a step references a specific table or section in a domain doc (e.g., "update the Policy Monitor Implementation Status table"), verify the exact heading/table name against the live file before writing the step. Section names drift between design intent and actual file. CC will find and update the correct location, but the discrepancy requires inference rather than clean execution. Fix: open the live file and paste the exact heading text into the spec.

**Domain doc steps -- confirm file path before writing step:** When a step specifies a file path marked "(to confirm)" or similar, COO must confirm the exact path before finalizing the prompt. CC will Glob to locate the file, but path ambiguity wastes a diagnostic round during execution.: business-calendar.md listed as "AOS/Domains/business-calendar.md (path to confirm)" -- actual path was `AOS/Domains/Compliance/business-calendar.md`. Required Glob mid-execution.

**Self-refactor routing:** If the target file is part of CC's permanent tax OR >15 KB with whole-file judgment required, the task does NOT go to CC. COO executes directly. See the routing table at the top of this file and `HAIOS/Architecture/C1-POST-MORTEM.md`. This is structural — no prompt engineering (safety gates, build log discipline, plan files, effort tuning) fully prevents the compaction storm mechanism. The only fix is to not route the task to CC in the first place. Evidence: C1 hit 6 compactions in 100 min on a task spec that predicted "compaction should not be needed"; partial completion increased workflow-task context tax by 68% before COO salvaged..

**Classification-heavy REFACTOR — require build-log table pre-ops:** For any REFACTOR that requires per-unit judgment (bullet classification, section categorization), the prompt §7 MUST require CC to write a classification table to the build log BEFORE mechanical operations. See "Classification Table Persistence" section above. Without this, compaction mid-task destroys the per-unit decisions and forces re-classification, burning the recovered budget. The table-on-disk pattern is compaction-insurance for judgment state, analogous to how the build log is compaction-insurance for procedural state.

**Bulk find-replace — replacement text and scope recon:** Before writing a bulk find-replace REFACTOR prompt: (1) Run scope recon — `grep -rl "OLD-TERM" "" --include="*.md" | wc -l` — to calibrate real file count. Manual inventories routinely undercount (Note: prompt listed 4 known files; actual was ~45). (2) Ensure replacement text does NOT contain the search term. If the replacement must reference the old name for context (archival notices), the term reappears in the post-task verification grep and produces false positives requiring a second cleanup pass. Use a self-contained replacement that omits the old name entirely. (3) Add explicit skip criteria in the prompt distinguishing "archival record references" (intentional — the Archived SSOTs table or changelog legitimately names the archived doc) from "active references" (to be replaced). "Expected: empty grep output" is an aspirational goal, not a hard pass/fail, when the Archived SSOTs table intentionally retains the doc name.

**ARCHIVE/CLEANUP prompt pattern:** For directory cleanup tasks that mix confirmed-safe moves with uncertain files, use a three-list structure: (1) MOVE LIST -- unconditional moves; (2) UNCERTAIN LIST -- files requiring a reference check before moving; (3) special-case files needing manual confirmation. For the UNCERTAIN LIST, include the exact grep pattern and the pending files to search, plus a binary decision rule (0 hits = move, any hit = hold + document). This structure eliminates judgment calls at runtime and produces a self-documenting build log. AP SCAN: check #55 (Drive path moves require Python shutil, not rm/mv).

**File-ops REFACTOR -- DoD counts must match routing table row counts:** When a routing table lists N source -> destination rows, the Definition of Done count for that group must match the table row count exactly. Mismatches (e.g., DoD says "20" but table has 19 rows) force CC to report against the wrong expected count and create false failure signals in the feedback file. Before finalizing a REFACTOR prompt: count the routing table rows for each group and write the exact count into the DoD checkbox. Never carry a count from memory or an audit report without re-counting the table rows in the prompt itself.

**Content-transplant REFACTOR — §SCHEMA-DIFF subsection required:** When a REFACTOR transplants prompt content from one container into another (Anthropic API templates → CC subagent files, MCP tool schemas → skill bodies, system prompts → agent definitions), the dispatched prompt MUST include a §SCHEMA-DIFF subsection explicitly listing each field-name diff the refactor introduces vs. preserves. Default: the source artifact's calibrated schema is SSOT; the prompt does NOT re-spec field names unless it provides a "schema-bump rationale" line. Rationale: v0.4 Path A migration prompt re-spec'd skeptic schema as `{local_id, verdict, reasoning, field_patches}` (carry-over from that session test schemas) vs source template's calibrated `{action, updated_fields, rationale}`. CC noticed only because Plan Mode CONTEXT QUERIES forced reading both. Without §SCHEMA-DIFF, CC writes either schema (both valid syntax) and the calibrated SSOT silently breaks. Companion drop: hedge phrasing like "(adjust if v0.X was actually labeled differently)" when CONTEXT QUERIES has already verified source state — hedges invite improvisation when the dispatched-prompt has the answer. Plan-time check for CC: in CONTEXT QUERIES for any transplant REFACTOR, read BOTH source artifact and new container's schema spec; surface divergence in Plan Mode before executing.

**INSPECT THEN DECIDE -- specify rename intent for kept file:** When a prompt tells CC to "keep whichever file has the higher version/date" and archive the other, include whether to rename the kept file to the canonical name. "Keep README-V2.md" is ambiguous: does "keep" mean leave at its current path (README-V2.md stays as README-V2.md) or does "keep" mean promote to canonical position (rename to README.md)? Without explicit instruction, CC defaults to leaving the file at its current path. If rename to canonical is desired, add: "rename kept file to {canonical-name} after archiving the other."

**File-path-ref routing entries — verify via grep before writing:** When a REFACTOR prompt's routing table specifies "update file-path references from `OLD-FILENAME` to `NEW-FILENAME`" in a target doc, run `grep -cE 'OLD-FILENAME' "{target-doc}"` at prompt-write time. If count = 0, the routing entry is a no-op — either omit it from the routing table or annotate it `"(0 hits confirmed — no-op)"`. Writing a routing entry based on assumption that the old filename exists in the doc leads to a false STOP (CC executes grep, finds 0 hits, wonders if it searched the wrong file).: ROADMAP routing table listed 2 file-path-ref updates; live grep found 0 hits — the ROADMAP uses shorthand labels (`P1-BLG-01`, `P1-BLG-05`) rather than filename links. Both entries were verified no-ops.

**DO NOT DISPATCH conditions — prefer DB/table-state over file-existence for sibling BLG sessions:** A DO NOT DISPATCH block that gates on `CC-PROMPT-BUILD-{SUBJECT}.md` EXISTS in a directory will falsely fail if the sibling CC session completed and archived its prompt (cc-task-prompts.md protocol moves prompt to `Archive/CC-Prompts/` on session end). Use Supabase table-existence check (`SELECT COUNT(*) FROM information_schema.tables WHERE table_name='{table}'` → 1) as the primary signal that a BLG build session completed. File-existence check is only reliable when the sibling session is still ACTIVE (prompt not yet archived). If gating on file: note "or confirmed archived at `Archive/CC-Prompts/S{N}-*`" in the DO NOT DISPATCH block.

**Frontmatter cleanup specs — note YAML format variations (ref):** When a REFACTOR prompt requires editing a YAML frontmatter field across multiple files (e.g., `related:`, `tags:`, `dependencies:`), check whether the target files use uniform format (all YAML list, all inline scalar) or mixed. If mixed, state explicitly in the prompt: *"Files may use either YAML list format (`field:\n - value1\n - value2`) or inline comma-separated scalar (`field: value1, value2`). Handle both formats."* evidence: REFACTOR-ARCHIVE-IS4AI-CLOSEOUT spec described `related:` cleanup without flagging the inline-vs-list split — C1 was YAML list, C2-C5 were inline scalar; CC discovered and adapted at runtime, but explicit pre-flighting eliminates the discovery step. Companion guidance: re-parse YAML after edit via `yaml.safe_load` to confirm validity regardless of which format was edited. Also: when removing a list entry, remove the leading `- ` and the trailing newline; when removing an inline-scalar entry, remove the comma + adjacent whitespace.
