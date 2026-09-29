# CONFIG EDIT Task Type

Surgical changes to config files, rules, claude.md, settings.

**Recommended effort:** `low / off`

- `off` for trivial single-value flips (one-line JSON change, env var set, boolean toggle)
- `low` for edits touching multiple keys or requiring surrounding-context awareness
- Unique among task types — `off` is default-safe here; rationale in `../PROMPT-TEMPLATE.md` `## Effort & Budget`

<!-- COO uses this file when writing a CC prompt for a CONFIG EDIT task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## SPEC

For each edit, provide:
- **File:** {full path}
- **Section:** {exact section heading as it appears in the file}
- **Insert after:** "{exact anchor line — enough context to be unique}"
- **Content:** {exact text to insert, verbatim}
```

## Session Learnings

**Config edit pattern (confirmed exemplar):** Specify (1) full file path, (2) target section name, (3) exact anchor line from the file, (4) exact content to insert. This gives CC an unambiguous match even if section ordering has shifted. Neighboring-line anchors prevent insertion at the wrong location when section names are similar.

**Surgical edits only:** Do NOT reformat, reorder, or rewrite existing content around the insertion. Preserve all existing content. Re-read the file after each edit to verify no corruption.

**Replacement/removal operations (first+last anchors):** When an edit replaces or removes a large block, provide both the first line (unique opening) AND the last line (unique closing) of the block as anchors. This gives CC an exact match in large files where similar phrases may appear. For playbook/doc strip tasks: include a "Do NOT touch these sections" list — negative space reduces the risk of unintended edits to adjacent valuable content.

**Naming discrepancy tasks involving COO/CC skill tables:** If a prompt asks CC to "fix a naming discrepancy" or "add a missing skill to the correct section," first verify the skill's current status: "Is this COO skill still active, or has it been superseded by a CC skill?" Prescribing "add to the active COO Skills table" vs. "add to EVOLVED/FOLDED" are different operations. Failure to ask causes a Plan Mode round-trip when CC discovers the skill is deprecated. Add to IMPORTANT NOTES: "Verify with Jordan whether {skill} is still in active use before placing it in the active vs. EVOLVED/FOLDED section."

**Cross-ref format in consolidation tasks:** When a consolidation task adds cross-references to a file that uses markdown tables, specify the format explicitly. Default: append `. Cross-ref: {file} {section}` to the Rule column of the relevant row. Without this, CC chooses the format and it may not match COO intent (footnote row vs. Rule column suffix vs. separate section).

**Batch-edit variant — stop criterion granularity:** When a CONFIG-EDIT task touches many files (e.g., 10+ frontmatter additions, bulk path updates), the default "stop and report" phrasing in §9 EXECUTION POLICY is ambiguous — CC may interpret it as halt-entire-batch on first structural exception, even when the correct behavior is per-file. For batch CONFIG-EDIT prompts, write §9 stop criteria explicitly:

```
§9 STOP CONDITIONS (batch variant)
(a) If any file's frontmatter is structurally unrecognizable: stop THAT file only, report as BLOCKED with the structural issue, continue to next file. Do NOT halt the entire batch unless all files are unrecognizable.
(b) If >50% of files are BLOCKED, halt the batch and report the full list before proceeding.
(c) Standard hard stops (data corruption, authentication failure, rm/delete safety) still halt everything.
```

 precedent: BUILD-Q2-D11 (19-doc `ssot-for` batch). Default §9 wording fired stop-the-batch on the first file where pre-asserted state was wrong (10 BestPractices files lacked `---` block contrary to prompt assertion). Plan Mode caught it; Jordan approved scope expansion. Writing the per-file stop variant explicitly would have prevented the Plan Mode round-trip. Apply to any batch-edit prompt touching 10+ files.

**Pre-computed Edit tool old_str/new_str for section inserts:** For prompts inserting a new section between two existing sections, pre-compute the complete `old_str` (anchor section body + following heading) and `new_str` (same content with the new section wedged between them) directly in the prompt's §7 CONFIG-EDIT Spec. This removes all Edit tool ambiguity — CC applies verbatim without interpretation. Especially effective for rule file edits where the insertion slot is bounded by two uniquely-named headings; the surrounding-context anchor pattern also handles any prior section-order drift.

**Rule-file edits and self-authorization:** When a CONFIG-EDIT prompt targets `~/.claude/rules/*.md` and includes a verbatim §SPEC with `old_str` / `new_str` content, the prompt itself IS the authorization. CC follows the standard Plan Mode flow (present plan, await Jordan's approval reply) and proceeds on approval — no separate Tier 1 chat confirmation is required. Codified in `~/.claude/rules/cc-task-prompts.md` `## Process-Feedback Loop` "Exception — COO-dispatched rule-edit prompts" paragraph. The HARD STOP applies only to CC-initiated rule edits surfaced during process-feedback, not to direct dispatched edits. Practical effect: COO does not need to write "this is Tier 1, awaiting separate approval" framing into rule-edit prompts.

**Conditional-halt revert plan:** When a CONFIG-EDIT batch contains an edit that (a) bumps a skill/agent version AND (b) is guarded by a conditional fallback (e.g., §1 ACCESS CHECK live DB check that halts the edit if a condition is met), include explicit revert instructions for any downstream lock-string references that assumed the edit would succeed. Format: in §7 SPEC or §8 IMPORTANT NOTES, add: "If Edit N is halted by [condition], revert [list of files / lock-string fields] to [prior values]." Without this, CC must make an undocumented corrective judgment call to restore system consistency — the corrective actions are logged in the build log but are not explicitly authorized by the prompt.: Edit 4 (proof-skeptic v1→v2) was halted; Edit 5 had already specc'd the lock string with `proof-skeptic v2`. CC reverted 4 downstream references (SKILL.md Pre-Flight lock, EXTRACTION-PROMPTS.md lock table row, assert_version snippet, changelog row) without explicit prompt authorization.

**Implementation skeleton success-gate assumption:** When a CONFIG-EDIT prompt includes an implementation skeleton for a multi-step script, the skeleton may assume a specific success gate (e.g., `if ($LASTEXITCODE -le 7)`) that doesn't match the actual script's error-handling pattern (e.g., an error-accumulator flag like `$hadError`). CONTEXT QUERIES must explicitly verify the error-handling structure. Add to IMPORTANT NOTES: "Verify the script's success gate before using the skeleton — loop-based scripts often use an error-accumulator flag; the metadata step must go inside the accumulator's `else` branch, not after a single command's exit code." The CONTEXT QUERIES gate will catch this at execution time, but flagging it explicitly reduces Plan Mode friction.

**§5 CONTEXT QUERIES — dual use for anchor-text pre-verification (-followup):** For CONFIG-EDIT prompts with multiple Edit-tool old_str anchors, populate `§5 CONTEXT QUERIES` with live reads COO performed at prompt-write time — in addition to (or instead of) semantic search queries. Record the exact observed anchor text for each edit: `"Read of {file} confirms {section} present; anchor line: '{verbatim text}'"`. This (a) satisfies (live state verified at authoring time), (b) surfaces stale anchors before dispatch — not mid-execution, (c) gives CC explicit confidence that old_str matches will succeed.-followup: §5 CONTEXT QUERIES documented 5 file reads with verbatim anchor confirmations → 0 old_str misses across 9 sub-edits. Complement to `Pre-computed Edit tool old_str/new_str ` — together they form the two-phase anchor contract: COO verifies anchors exist (§5), then provides verbatim pairs (§7 SPEC).

**Workflow node CONFIG-EDIT — include node type in spec:** When a CONFIG-EDIT or BUILD prompt targets specific n8n workflow nodes, the spec must include the **node type** (Code / Switch / Postgres / HTTP Request / etc.) alongside the node name. Without this, CC may look in the wrong place — e.g., searching the Switch conditions for a threshold that actually lives in an upstream Code node. Add a "Node type" column to any per-change spec table that targets workflow nodes. Example: `| Classify 3P Signals | Code | Add taxonomy block to SYSTEM_PROMPT |` vs. just `| Classify 3P Signals | Add taxonomy block |`. The node type tells CC whether to look in `jsCode`, Switch `rules`, or node `parameters` — saves one discovery round. pre-assertion check is the safety net, but explicit node type is the prevention.

**Version bump — distinguish YAML frontmatter vs inline version block:** When specifying "Frontmatter bump: X → Y" in a CONFIG-EDIT spec, verify at prompt-write time whether the target file uses a YAML frontmatter block (`---` delimited, `version: X.X`) or an inline version field (`**Version:** X.X` in the body). Files without YAML frontmatter (e.g., `CONTINUOUS-LEARNING.md`) require bumping the inline field — specifying "frontmatter bump" is technically incorrect and may confuse CC. Preferred phrasing for inline-version files: "Version field bump: `**Version:** X.X` → `**Version:** Y.Y`; add `**Updated:** {date}` on the following line." gate: include the file's version format in `§5 CONTEXT QUERIES` live read.

**Multi-target GET-all-before-any-PUT:** When a CONFIG-EDIT targets the same field/value across N workflows, instruct CC to GET all N targets first and verify the current field value before writing ANY PUT. Build a fix list — only targets where current value ≠ target value get a PUT.: 5 of 6 blog workflows already had the correct Slack node config; only BLG-08 needed the channel change. 5 unnecessary PUTs avoided. Instruction to add in §4 TASK: "GET all target workflows. For each, confirm [field] current value. Fix list = workflows where current ≠ target. Do NOT PUT any workflow not on the fix list."

**Rule-file edits (`~/.claude/rules/*.md`) — Edit tool blocked by classifier:** The safety classifier blocks the Edit tool on files under `~/.claude/rules/`. CC cannot apply these edits autonomously — the block fires regardless of Plan Mode approval. **Workaround:** pre-write a Python `open.read/str.replace/write` script in the prompt and add a note for Jordan to run it via `! py C:/path/to/script.py`. Pre-baking the script removes the mid-task interruption. Companion to that session (Jordan authorization gate for rule-file edits) — covers authorization; this caveat covers the tool-level block.

---
