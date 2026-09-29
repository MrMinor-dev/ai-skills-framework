# CC Prompt Template

Universal template for all CC task prompts. Prompts contain ONLY task-specific content. Repeating protocols (secrets, feedback) live in global CLAUDE.md — CC already has them.

## Structure

```
Recommended model: Sonnet | Haiku | Opus
Recommended effort: low | medium | high | xhigh | off
Recommended budget: {time, iterations, or token cap — 2-3x competent-engineer estimate}
/goal {acceptance_condition} — verbatim; Jordan types this as the FIRST message in the CC session before the file one-liner
1. CC ACCESS CHECK      (always — verify access before starting)
2. RESOURCES            (always — what this task reads and writes)
3. AP SCAN              (always — anti-patterns checked, visible in prompt)
4. TASK                 (always — what to do)
5. CONTEXT QUERIES      (always — semantic search before planning)
6. CONTEXT              (always — what CC needs to know)
7. TASK-TYPE SECTION    (pick one — detailed spec for the task type)
8. IMPORTANT NOTES      (always — gotchas, constraints)
9. EXECUTION POLICY     (always — stop criteria, fallback, verification method)
10. DEFINITION OF DONE  (always — checkboxes)
```

**What's NOT in prompts (lives in global CLAUDE.md instead):**
- Secrets Protocol — CC reads this every session from `~/.claude/CLAUDE.md`
- Feedback Protocol — CC reads this every session from `~/.claude/CLAUDE.md`
- Feedback file template — CC knows the structure from `~/.claude/CLAUDE.md`
- Archive instructions — CC knows the pattern from `~/.claude/CLAUDE.md`

**Why:** Long prompts cause CC to miss end-of-file instructions (~70% adherence on prompt content, degrades with length). Moving repeating sections into CLAUDE.md (also ~70% but reinforced every session) keeps prompts short and focused on task-specific content. Critical protocols should continue migrating UP the reliability hierarchy (hooks > rules > skills > CLAUDE.md > prompts) as we build out Phase 3.

---

## Pre-Assertion Verification Rule

**Rule:** Any prompt asserting state of a target file (field value, version number, frontmatter presence, structural claim, finding count) must verify the assertion via a **live file read at prompt-write time** — not from prior-session notes, prior-audit summaries, or memory.

**Why:** (n=3+ Q2-2026) — assertions sourced from prior-session notes are stale by the time CC executes, causing reclassification mid-task, wasted iterations, or incorrect execution. Files change between sessions; subagent summaries can hallucinate; even self-authored Phase N notes can drift by Phase N+1.

**Enforcement:**

1. **At prompt-write time:** COO reads target files via `Filesystem:read_text_file` BEFORE writing assertions. The read output becomes the assertion basis.

2. **In the prompt's CONTEXT QUERIES section:** explicitly list the file reads that verified each assertion. **The session number is mandatory in every CONTEXT QUERIES entry — omitting it breaks traceability.** Use the header format `CONTEXT QUERIES (verifies assertions below, all reads):` where is the current session. Example:
   ```
   CONTEXT QUERIES (verifies assertions below, all reads):
   - Read of HAIOS/Knowledge/DIGITAL-ASSETS.md confirmed field is `ssot: true` at line 3 (NOT `ssot-for: true` as the audit notes claimed)
   - Read of 19 target files: 9/19 have `---` YAML frontmatter; 10/19 use body-only `**Version:**` format
   ```
   **COO self-check:** if any CONTEXT QUERIES entry is missing `` → add it before dispatching. (n=3 prompts dispatched without session number.)

3. **Make assertions conditional** when the value may drift between prompt-write and execution:
   - Avoid: "Bump version from 1.2 to 1.3"
   - Prefer: "Bump version: read current, IF current is 1.2 THEN set to 1.3 ELSE STOP and report discrepancy"

4. **AP SCAN section must list** on any prompt that contains state assertions about target files.

**Self-check before dispatching:** if the prompt contains "the file HAS X" / "VERSION IS X" / "FIELD Y EQUALS Z" without a corresponding CONTEXT QUERIES entry from THIS session — STOP and re-verify. Prior-session verification does not count; state may have drifted.

**Applies to:** all task types. Especially high-leverage on BUILD, FIX, REFACTOR, CONFIG-EDIT, and multi-phase AUDIT tasks where Phase N claims propagate into Phase N+1 prompts.

---

## Effort & Budget

Every CC prompt specifies effort (thinking depth) and budget (consumption cap). Both live in the prompt header alongside Recommended model.

### Effort

**Value definitions (SSOT):** `HAIOS/Knowledge/BestPractices/CC-DEVELOPMENT-PLAYBOOK.md §Token Optimization §Effort Parameter` owns per-value semantics (low/medium/high/xhigh/max) and the MRMINOR task-class default table. Do not duplicate those here (Pattern B per SSOT-REGISTRY Cascade Rule Q2 audit D6).

**Prompt-authoring-specific notes (retained here):**

- **Defaults** live in each task-type file (§7); override per-instance when the specific task deviates from the class norm.
- **`off` is a prompt-authoring shorthand** meaning "disable thinking entirely" — not one of CC's API effort values (which stop at `low`). Default-safe only on CONFIG-EDIT. Elsewhere it's an explicit opt-in for tasks with no meaningful reasoning to do (classification, extraction, formatting, simple rewrites, tagging). Other task types bottom out at `low`.
- **Why this slot exists (Opus 4.7):** CC's default effort is `xhigh`. Without an explicit recommendation, trivial tasks get xhigh-level thinking — the biggest invisible cost source on 4.7 (`Archive/HAIOS/OPUS-4-7-BEST-PRACTICES.md` Tracker #5, archived). The slot closes that gap.

### Budget

Starting rule: **2–3x what a competent engineer would need** (from Anthropic's agentic-workflow checklist). Expressed as time, iterations, or token cap.

**Examples:**
- `Recommended budget: 30 min` (simple config edit)
- `Recommended budget: 2 hrs` (workflow build + verification)
- `Recommended budget: 3 iterations max` (debugging task)
- `Recommended budget: 20k tokens` (research synthesis)

Budget is a guardrail, not an accurate estimate. When CC approaches the cap, CC stops and asks rather than burning through. Undersetting forces earlier check-ins; oversetting invites scope creep.

**Form:** `Recommended budget: {value}` in the prompt header, alongside `Recommended model:` and `Recommended effort:`.

### Thinking-steering snippets

Copy-ready phrases for steering 4.7's adaptive thinking when effort alone isn't enough signal:

- **More thinking:** "Think carefully and step-by-step before responding; this problem is harder than it looks."
- **Less thinking:** "Prioritize responding quickly rather than thinking deeply. When in doubt, respond directly."

Use sparingly. If effort + budget are set correctly for the task, these shouldn't be needed — they're a fine-tuning tool, not a primary lever.

---

## Subagents

4.7 prefers doing work in one response unless explicitly told to fan out. Include subagent guidance in prompts where parallelism or output-isolation matters.

**Mental test before recommending subagent use:** will CC need the full tool output again, or just the conclusion? If just the conclusion → subagent (keeps intermediate noise out of main context). If full output → do it in main context.

**Copy-ready negative pattern** (Anthropic's default guardrail):
> Do not spawn a subagent for work you can complete directly in a single response. Spawn multiple subagents in the same turn when fanning out across items or reading multiple files.

**Copy-ready positive patterns** (Thariq, Anthropic):
> Spin up a subagent to verify the result of this work based on the following spec file.
>
> Spin off a subagent to read through this other codebase and summarize how it implemented the auth flow, then implement it yourself in the same way.
>
> Spin off a subagent to write the docs on this feature based on my git changes.

**Where to put subagent guidance in a prompt:** IMPORTANT NOTES (§8), unless the subagent use is central to the TASK itself (§4) or part of the task-type spec (§7). For AUDIT prompts that launch parallel Explore agents, the guidance typically lives in CC ACCESS CHECK (§1) or TASK-TYPE SECTION (§7 — AUDIT.md already documents the pattern).

**Custom subagent registration is fixed at session boot.** If the prompt's TASK both writes a new `~/.claude/agents/*.md` file AND requires invoking that subagent in the same task, the invocation will fail with `Agent type '{name}' not found` because the `subagent_type` enum doesn't refresh mid-session. **Mitigation:** split into install-prompt + use-prompt (operator restarts CC between), or pre-stage the agent file out-of-band before the prompt runs. For TEST prompts probing new subagent infra, add a §1 CC ACCESS CHECK bullet: "subagent_type `{name}` is already in the Agent tool's enum at session start." See the anti-pattern log.

---

## Calibration Target (evaluator / skeptic / judge prompts only)

For prompts whose output is a verdict on other prompts' output (skeptic, judge, rubric-scorer, classifier-evaluator), state the empirical band the prompt should land in across realistic batches. This is a load-bearing acceptance criterion — it lets CC verify against the sample outputs, and it forces the metric to be defined precisely up front.

**Form:** add `Calibration target:` as a top-level field in the prompt header alongside Recommended model / effort / budget. Define the metric, the action set it counts, and the band:

```
Calibration target: skeptic downgrade-or-flag-with-updates rate within 5–15% across batches of 10+ rows; <5% = too lenient, >15% = upstream extractor too generous.
```

**Define the metric precisely.** "Downgrade rate" alone is ambiguous when the action set includes multiple non-accept actions (downgrade, flag, reject). Either expand the metric to count all field-mutating actions, or rename the actions for clarity.: the SKEPTIC-PASS-PROMPT-v0 task hit Round 2 "0% downgrade" while emitting 1 flag-with-updates and 1 evidence-flag on 6 rows — metric definition hid real activity.

**Iteration contamination warning.** If the prompt iterates on the same sources used to discover the calibration gap, the post-iteration calibration figure measures rule-fix effectiveness on those exact rows, not generalization. The real calibration data comes from the first novel batch. Document this in IMPORTANT NOTES so the round-2-on-same-source figure isn't taken as the prompt's true calibration..

---

## Known Caveats (optional)

For prompts that touch known-friction surfaces (hook-blocked installs, expired credentials, particular shell quirks, schema ambiguities), include a brief callout in the prompt. Saves CC from rediscovering the friction mid-execution.

**Form:** as a sub-block within IMPORTANT NOTES (§8) or as its own callout above EXECUTION POLICY (§9):

```markdown
## KNOWN CAVEATS
- Anthropic SDK install is `safety-guard.ps1`-blocked; either pre-install via `! py -m pip install anthropic` before the task, or use stdlib `urllib.request` in test scripts (identical API output for tool_use calls).
- {one-line description of caveat + standard workaround}
```

Reserve for genuine known-friction items. Don't pad with hypothetical caveats — the section earns trust by being short and accurate.

**Standard caveats to include when relevant:**
- **FIX / MIGRATE / BUILD + DB DDL tasks:** **Probe psycopg2 in §1 ACCESS CHECK; never assume it is unavailable.** Write a throwaway script that `open`s the connection string, connects, runs `SELECT 1`, and exits (see §1 for why `py -c` with a `.secrets/` path is blocked), then branch on the result: **connects → PATH A** — CC runs the migration and the write-assertion smoke tests autonomously; **fails → PATH B** — surface the SQL to Jordan for the Supabase SQL Editor at https://supabase.com/dashboard/project/{{SUPABASE_PROJECT_REF}}/sql. Record which path fired in the build log. **This replaces the flat "Supabase port 5432 is unreachable — psycopg2 DDL will fail at connection" caveat** (Confirmed 2026-05-04; FIX-TRANSCRIPT-EMBEDDINGS-RLS): `BUILD-AGENCY-SUPPLY-TABLE` probed, connected on the first try, and ran a 6-step DDL migration idempotently with zero Jordan touches. See the anti-pattern log's amendment for why that caveat stood many sessions unchallenged while and simultaneously *required* psycopg2 — **a "don't" nobody tests is never contradicted.** The probe is one call and it is the only honest answer; do not replace one assumption with its opposite. **Postgres connection string file: `.secrets/supabase-db-connection.txt`** (confirmed — NOT `supabase-postgres-url.txt`; using the wrong filename causes a FileNotFoundError that looks like a secrets path issue). **DDL stop-hook risk:** If the `/goal` condition requires actual DB state AND Jordan does not confirm execution within the session, the stop-hook fires every turn CC responds. Either anchor `/goal` to an outcome CC can reach alone ("migration SQL written and surfaced to Jordan" — matches the Execution Policy stop criteria), or include in §8 IMPORTANT NOTES: "If Jordan does not confirm SQL execution, use the n8n DDL workflow path as fallback — CC-autonomous per CLAUDE.md." **A `/goal` anchored to a Jordan action is a `/goal` CC cannot close.**
- **BUILD prompts targeting code/infrastructure with partial existing implementation:** Include in §6 CONTEXT a "Current Implementation State:" line that lists what already exists vs. what is specifically missing. Without this, CC must infer build-from-scratch vs. surgical-add, often misreads the gap, and rebuilds working code. Form: "The file already contains {list what's there} — missing pieces are {list absent items}." (Note: CF Worker already had 4-threshold structure; prompt said "currently fires kill-switch only at cap-breach" — CC rebuilt existing structure from scratch before discovering what was actually missing.)
- **BUILD/FIX tasks that insert a node mid-pipeline:** "Inserting a new node between Node A and Node B replaces Node B's `$input` — Node B will receive the new node's rows, not Node A's original output. If Node B currently reads signals/data from `$input.all` or `$input.first`, update it to use `$('Node A Name').first.json` (cross-node upstream ref) instead. Cross-node refs to pre-insert nodes are always accessible per N8N-API-REFERENCE.md pitfall #44." (Note: Fetch Disposition History insertion, Normalize Signals upstream ref fix.)
- **FIX / CONFIG-EDIT + workflow Code node (jsCode replacement via Python):** "n8n stores Unicode chars in jsCode as literal 6-char escape sequences — `•` (U+2022) is stored as `•`, not the actual bullet character. Python string matching using the actual Unicode char will silently fail (`OLD not in js_code`). Use `\\u2022` (double-escaped) in Python OLD/NEW strings to match the literal sequence. This applies to ALL Unicode escapes including emoji surrogate pairs (e.g., `\\ud83d\\udd0e` for 🔎, `\\u00b7` for ·). Debug with `repr(js_code[idx:idx+80])`. See the anti-pattern log and N8N-API-REFERENCE.md Code Node section." (Note: FIX-LANDSCAPE-SUMMARY-DEDUP — first script used actual bullet char, failed; second used `\\u2022`, succeeded.: CONFIG-EDIT-DIGEST-LIVENESS-LINE — emoji/Unicode in liveness line; same fix applied.)
- **BUILD workflow with Slack Events API trigger:** Before writing the prompt, verify: (1) a Slack App exists with this workflow's webhook URL registered in Event Subscriptions — if not, create one first (Jordan action: create app → add bot to channel → add scopes: `chat:write`, `channels:history`, `channels:read`, plus `groups:history`/`groups:read` if private channel; Event Subscriptions: `message.channels` and/or `message.groups`); (2) the n8n Slack credential for that app exists and shows Connected; (3) the bot is added to the target channel. One Slack App = one registered Event Subscriptions URL = one workflow receives events. If multiple workflows need events from the same channel, they must share one URL via a routing workflow — OR each workflow gets its own Slack App. Never assume an existing Slack App covers a new channel/workflow. (BLG-06a-v2 + BLG-09 blocked by this gap.) "Two required checks before building: (1) verify the `slack_channel` DB column contains IDs not names — run `SELECT slack_channel FROM {table} LIMIT 3`; if the value starts with `#`, it's a display name and will cause `channel_not_found`; fix the data or the source workflow first. (2) Use the same Slack credential type as the originating workflow — if the source workflow used `slackOAuth2Api`, this workflow must also use `slackOAuth2Api` (bot tokens require explicit channel invitation; OAuth2 user tokens inherit membership). Add to §1 CC ACCESS CHECK." (Note:)
- **FIX + workflow node removal (node referenced by README internal ID):** "The prompt's node reference (`{readme_id}`) may be a README authoring shorthand — the live n8n display name may differ. Add a §LIVE STATE VERIFICATION gate: 'Confirm live display name of `{readme_id}` node from GET before deletion; use live display name for all node-removal and connection-removal operations. If display name differs, note the discrepancy in the build log.' Scripts that search by README shorthand will silently no-op if the display name differs." (Note: prompt specified `query_quarantine`; live display name was `Query Quarantine Emails`.)
- **FIX tasks in a D-cascade (task depends on prior D-task having modified a file):** For each file a prior D-task claims to have modified, add a CONTEXT QUERY: "Re-read `{file}` and confirm `{expected change}` is present before proceeding -- do not trust the task note." Without this, CC accepts the note at face value and may run against stale state. Evidence: FIX-CC-CONFIG-P15-STALE-REF-RESOLVER -- D2 claimed to strip all refs to `CC-FEEDBACK-TEST-PROJECT-RULES.md`; only CLAUDE.md was stripped; MANIFEST.md still held the ref; DoD said "no longer emit" but the live-ref gate still found it. Root cause is -- prior-task-note is not a live verification.
- **BUILD site prompts with parametric variants (hero variant, card count, email mode, CTA URL, etc.):** Add a `## Dispatch Parameters` block immediately after the prompt header, listing each variant explicitly (e.g., `Hero: primary`, `Cards: 5`, `Footer email: {{EMAIL}}`). If dispatch parameters appear only in the `/goal` line or deep in footer instructions, CC must infer them — inference creates ambiguity when multiple valid options exist. Dispatch Parameters block is the authoritative source. (Note: all 5 variant decisions inferred from `/goal` line — worked, but explicit block would have removed the inference step.)
- **BUILD workflows that write files to a GitHub repo via PAT (GitHub Contents API PUT):** If Jordan creates a Fine-grained PAT, it needs **Repository permissions → Contents → Read and write** — not just "code" (Read) access. Classic PAT needs `repo` scope. Add to §1 CC ACCESS CHECK: "Verify `github-token.txt` exists. If creating a Fine-grained PAT: Repository permissions → Contents must be set to **Read and write** (not just Read) — required for `PUT /repos/.../contents/...`. Without it: 403 'Resource not accessible by personal access token'." (Note: PAT had Contents:Read-only; 3 blocked executions before correct scope identified.)
- **BUILD workflows that push to a GitHub repo (any Git Data API write, `PATCH /git/refs/...`):** Add to §10 DoD: "Post-activation git audit: within 5 minutes of activation, run `gh api repos/{org}/{repo}/commits?per_page=5` and confirm no unexpected automated commits. Any `rollback:` or automated commit message = deactivate immediately and investigate." Also add to §8 IMPORTANT NOTES: "Confirm all upstream callers of this workflow have bug-free failure-detection logic before leaving both active — a false-failure in any caller that calls this workflow creates a real git commit." Commit message format should always include `Run: {{ $execution.id }}` for post-mortem traceability. (Note: BLG-07 called Site-Restore 4 min after a real publish because BLG-07's verification was also broken — real revert commit `2eea7fd5` created on a live post.)
- **BUILD/FIX prompts with `httpHeaderAuth` nodes (GitHub PAT, external APIs):** NEVER inject `httpHeaderAuth` credential by type alone — multiple `httpHeaderAuth` credentials exist for different services. Specify exact credential name AND ID in the prompt. Add to §1 ACCESS CHECK: "Run `GET /api/v1/credentials` and list all `httpHeaderAuth` credentials. Confirm the correct credential for this service by name — do NOT use the first result." Evidence: Site-Restore build injected `{{CREDENTIAL}}` onto all 7 GitHub API nodes; every execution 401'd for months. GitHub PAT credential name/ID must be verified at prompt-write time. See CLAUDE.md httpHeaderAuth section.
- **BUILD workflow prompts that INSERT into shared AOS tables (`haios_workflow_registry`, `agency_blog_topic_queue`, etc.):** Include the §6 CONTEXT → Schema verification block listing each table's CHECK constraint values — even for BUILD prompts (not just FIX). These tables have non-obvious constraints: `haios_workflow_registry.layer` must be lowercase (`aos`, `haios` — NOT `AOS`); `agency_blog_topic_queue.category` must be one of `thought_leadership|stack_walkthrough|pain_row|seo`. Constraint lookup: `SELECT cc.check_clause FROM information_schema.check_constraints cc JOIN information_schema.table_constraints tc ON cc.constraint_name = tc.constraint_name WHERE tc.table_name = '{table}' AND tc.constraint_type = 'CHECK'`. Add a §1 CC ACCESS CHECK bullet: "Run schema pre-check on every table this workflow INSERTs into." (Note: layer uppercase and wrong category caused two failed INSERTs mid-build that a pre-check would have prevented.) See the anti-pattern log. |
- **AUDIT/FIX prompts scoping a field-removal edit to specific named nodes (e.g. "remove field X from Node A and Node B"):** If the prompt author has not already grepped the live node set to confirm the field actually appears in every named node, add to §1 CC ACCESS CHECK: "Grep the full live node set for `{field}` before editing — confirm which named nodes actually contain it. If a named node has zero occurrences, report it as a no-op in the DoD, don't skip the check." Without this, CC either wastes a verification cycle discovering the no-op mid-edit or (worse) infers presence from the prompt's assertion alone (class). (Note: prompt named `Parse Response` for newsletter-field removal alongside `Build Prompt`; live grep showed 0 occurrences in `Parse Response` — the field was Build Prompt-only. DoD's "any other occurrence reported, not removed" caught it, but a pre-grepped scope would have saved the verification step.)
- **BUILD+workflow prompts with Code nodes (any mode, n8n Cloud):** The following four fields are **required** in every BUILD+workflow prompt that includes a Code node. Omitting any of them forces CC to guess or inspect mid-build, causing re-patch cycles (Note: 5 Code nodes re-patched twice due to +). Add as explicit named fields in §6 CONTEXT or §7 task-type spec:
  1. **Code node mode:** always `runOnceForAllItems` on n8n Cloud. Never omit — default is `runOnceForEachItem`, which: (a) crashes with JsTaskRunnerSandbox realm isolation bug AND (b) silently produces wrong output in nodes using `$input.all` — N items → N runs of 1 item each instead of 1 run of N items. Both failure modes are invisible without testing. AP SCAN must check + on every prompt with Code nodes.
  2. **Drive folder parent ID:** explicit folder ID (not name) if the workflow creates Drive folders.
  3. **Slack channel ID:** explicit channel ID (not `#name`) if the workflow posts to Slack.
  4. **Input Source Substitutions table (port-from-OLD only):** list every `$('NodeName')` reference that changes from the OLD workflow to the new one, with its replacement. Required any time the prompt says "port from" or "based on" an existing workflow. Without this table, CC must inspect the OLD workflow mid-build to guess substitutions. Format:
     ```
| OLD reference | New reference | Node context |
|---|---|---|
| $('Old Node').item.json | $('New Node').item.json | Resolve Business Code node |
     ```

---

## Iterative Test Output Paths

For prompts expected to iterate prompts/samples across 2+ rounds (extractor refinement, prompt-tuning tasks, evaluator calibration): parameterize SAMPLE-OUTPUTS paths with a round suffix so iteration history is preserved as evidence. Overwriting prior-round outputs at the same path loses the calibration story.

**Pattern:**

```
SAMPLE-OUTPUTS/
├── {source-id}-round1.json
├── {source-id}-round1-skeptic.json
├── {source-id}-round2.json
└── {source-id}-round2-skeptic.json
```

The DoD checkbox should reference the final round (`round2.json`) but the build log captures the round-1 → round-2 delta as the calibration narrative. Without round-suffixed paths, the only evidence of what changed lives in the build log prose — brittle, not auditable post-task.

---

## COO PRE-WRITE GATE

**Before writing any prompt, COO must verify these gates. This is not optional — skipping them causes CC to burn fix cycles on problems the prompt introduced.**

| Gate | Trigger | Action |
|---|---|---|
| **Schema verification** | Prompt contains ANY SQL, Supabase REST filter, DB column reference, or DoD/test queries that name specific tables | Use live safe-sql query to verify every table name, column name, and constraint value BEFORE writing into the prompt — including DoD test queries. The live DB is authoritative (archived — live DB is authoritative). For column names: `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '{table}'`. For check constraints: **do NOT query by constraint_name pattern** — constraint names rarely contain the table name (e.g., `valid_layer` on `haios_workflow_registry`). Use: `SELECT cc.constraint_name, cc.check_clause FROM information_schema.check_constraints cc JOIN information_schema.table_constraints tc ON cc.constraint_name = tc.constraint_name WHERE tc.table_name = '{table}' AND tc.constraint_type = 'CHECK'`. **safe-sql truncation caveat:** the safe-sql markdown response silently truncates long CHECK `check_clause` / ARRAY definitions mid-literal — no error, no indicator (N8N-API-REFERENCE Pitfall #31 sub-note). If the result ends with `'value'::c` or an incomplete ARRAY, it is cut. **Whenever a prompt inserts a NEW value into a platform/domain/type-keyed shared table, verify the full allowed set via psycopg2 direct** `pg_get_constraintdef(oid)` (`SELECT pg_get_constraintdef(oid) FROM pg_constraint WHERE conrelid='{table}'::regclass AND conname='{constraint}'`) — confirming the column exists is NOT sufficient; the new key value may be absent from the CHECK set and fail at INSERT (Note: `chk_cb_platform` on `aos_circuit_breaker` blocked `cc-config-loop` mid-build; required Jordan DDL). Never write column values from memory or docs.: guessed `'HAIOS'` for layer — actual value is `'haios'`. Constraint query by name pattern returned 0 rows; table join returned the `valid_layer` constraint. **safe-sql URL must always be the canonical endpoint `https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}}` — never a workflow-specific webhook ID (e.g., `{{WORKFLOW_ID}}`). Workflow-specific IDs go inactive when their workflow is deactivated.**: prompt used `{{WORKFLOW_ID}}` (inactive, 404); canonical URL worked. **When column names are uncertain, always include an explicit CC ACCESS CHECK step: "Run `SELECT column_name FROM information_schema.columns WHERE table_name = '{table}' ORDER BY ordinal_position` via safe-sql to confirm before writing SQL."**: prompt flagged `topic_name` uncertainty without requiring live verification — CC had to derive `title` from live schema mid-build. |
| **Anti-patterns scan** | Every prompt | Use the TASK-TYPE INDEX in `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then; `references/ANTI-PATTERNS.md` is a redirect stub) to check relevant APs. Record results in the prompt's `## AP SCAN` section (section 3). An empty AP SCAN = an incomplete prompt. |
| **Existing file check** | Prompt references a Drive file CC will read or edit | Verify the file exists and the path is correct via `Filesystem:read_file` or `Filesystem:list_directory`. For anti-pattern graduation tasks: verify each "Graduates To" target file exists at the specified path before including it in the graduation table. Mark inaccessible targets as "Skip if not found" — do NOT leave CC to infer. **When spec says "use the existing format/pattern in file X" or "follow the registration pattern in file X": verify the referenced format or section actually exists in the file before spec-ing that action.** D#9: spec pointed to DIGITAL-ASSETS.md saying "use the same registration pattern as other SSOT-Y docs in the registry" — file has no doc registry section; CC correctly skipped and surfaced for COO guidance. |
| **Resource conflict check** | Dispatching 2+ CC tasks in parallel | Compare RESOURCES sections across all concurrent prompts. If any WRITES overlap → sequence the tasks (finish first, then start second). If READS overlap with another task's WRITES → also sequence. No overlap → safe to parallelize. |
| **Workflow ID verification** | Prompt references any n8n workflow ID | `GET /api/v1/workflows/{id}` and confirm the `name` field matches the expected workflow. Record id / name / version / active in the prompt's `### Live State Verification` subsection (§6 CONTEXT). Workflow IDs go stale when workflows are rebuilt. A wrong ID causes CC to fail on the first API call and requires name-based recovery (wasted round-trip).. |
| **Workflow node count** | Prompt is FIX, BUILD (existing workflow), or CONFIG-EDIT on a workflow node | While doing the Workflow ID GET above, record the live node count in the prompt's `### Live State Verification` subsection (§6 CONTEXT) as `VERIFIED NODE COUNT: {n}`. Never carry counts over from memory, READMEs, or prior prompts — workflows gain nodes silently. Use it in the DoD halt condition: "node count = {n} after fix" (or {n+added} if nodes are being added). Two count-drift failures in that session (Remediation Trigger: 25→26, Daily Digest: 19→20) — both preventable with this gate.. **Mechanical + relative + never-in-`/goal`:** derive `{n}` as the LENGTH of the GET `nodes` array — never eyeball-scan the node list (eyeball miscounts recur: §9 prompt said 28/live 26; §10 prompt said 13/live 12). For no-node-delta FIX/CONFIG-EDIT, write the DoD halt **RELATIVELY** — "node count unchanged from pre-fix GET" (CC derives pre+post from its own GETs) — NOT an absolute COO-authored "== {n}"; a COO miscount of an absolute number falsely blocks a correct fix. **NEVER put an absolute node count in the `/goal` completion condition** — it is a DoD guardrail, not a completion anchor; a wrong count there trips CC's stop-hook on correct work. §10: `/goal... node count still 13` blocked a verified-correct fix; live was 12. |
| **Node operation type** (FIX only) | Prompt targets a specific node's config or expression | `GET` the workflow and confirm the target node's `operation` field (`executeQuery`, `insert`, `select`, etc.) matches the fix spec's assumption. Nodes can be refactored between sessions (e.g., `insert` → `executeQuery` with column-mapping). Record `operation: {value}` in the `### Live State Verification` Node-level block.: fix targeted `operation: insert` on a node that had been refactored to column-mapping `executeQuery` — wrong shape entirely; actual value source was an upstream Code node. |
| **Node existence for "+N nodes" specs** | Prompt specifies adding N nodes | `GET` the workflow and verify each proposed node doesn't already exist by name. List in the `### Live State Verification` Node-level block as ADDED (new) vs UPDATED IN PLACE (already exists). A "+2" spec that's actually +1 new + 1 update fails on naive execution.: "+2 nodes" estimate ignored 1 already-present node; actual delta was +1. |
| **Upstream node output shape** (BUILD with cross-node reads) | Prompt adds a node that reads from an existing Code node via `$('NodeName').all` or `.first` | `GET` the workflow and read the source Code node's full `return [{ json: {...} }]` statement. Verify the target field is in the returned object — NOT just used internally as a `const`. A Code node may compute `allFindings` internally without exposing it in the return. Frame the check as "output structure" not just "field names.": spec assumed `$('Compile Report').all` returns per-finding items; actual output was one summary item; `allFindings` was internal-only. Fix: added `allFindings` to the return (additive).. |
| **HC component name** | Prompt references or writes a `haios_health_checks` `component` value | Source the component string from the workflow's actual HC node SQL via `GET` — NOT from memory, docs, or naming-convention guess. Record `component: '{name}'` in the `### Live State Verification` subsection.: prompt used `HAIOS-Finance-…`; the workflow's actual HC node SQL used `AOS-Finance-…` — every verification query returned empty until fixed. |
| **Symptom freshness (LAST VERIFIED)** | FIX prompt describes one or more current bugs / symptoms | For each symptom, confirm it's still reproducible at prompt-authoring time (webhook replay, safe-sql detection query returning the expected failing row, or HC row inspection within last 24h). Record `LAST VERIFIED: {YYYY-MM-DD} (execution ID: {id} / HC: {row evidence})` next to each symptom in the `### Live State Verification` subsection. Bugs can be fixed between prompt authoring and CC execution.: prompt listed Bug B1 (ExpressionError) as current symptom — had been fixed in a prior session. CC chased a non-existent bug for half a cycle. |

| **Routing table location** | Prompt has a `## COO ROUTING DECISIONS` section populated by COO, AND references an external research/audit report that contains its own blank routing table | Add a callout immediately above the routing table in the prompt: `> **CC: decisions are in the table below in THIS file — not in the referenced report. The report's routing table is a blank template only.**` Without this, CC may open the report, find its blank table, and stop asking COO to fill it in — even when the prompt's table is fully populated.: Ph2-Ex and Ph5-Ex both hit this failure; CC stopped on first dispatch, re-dispatched with populated table, CC then executed correctly. |

| **DO NOT DISPATCH conditions — sibling CC sessions** | Prompt has a DO NOT DISPATCH block gating on a sibling CC task's output file existing | Prefer table/DB-state check as primary signal (e.g., `SELECT COUNT(*) FROM information_schema.tables WHERE table_name='{table}'` → 1) over file-existence check. Sibling CC sessions archive their prompt files on completion (cc-task-prompts.md protocol) — a file-existence check will falsely fail after the sibling archives. If gating on file: add fallback "(or confirmed archived at `Archive/CC-Prompts/S{N}-*`)" in the DO NOT DISPATCH block.: dispatch condition checked for `CC-PROMPT-BUILD-AGENCY-BLOG-TOPIC-QUEUE.md`; file was archived post-BLG-01 completion; caused false-alarm during §1 ACCESS CHECK. |

** lessons:**
- *Schema gate:* COO wrote `?component=in.(...)` for `haios_remediation_log` DELETE — table uses `workflow_name`..
- *Resource conflict:* 3P Collector error fix + cron rhythm fix both WRITE to workflow `{{WORKFLOW_ID}}`. Second PUT would overwrite first. Caught manually, but RESOURCES section would have flagged it at dispatch time.

**DB-schema self-check before dispatching:**

Before signaling the prompt ready, scan the prompt for every DB entity name (table, column, constraint, enum type, function, trigger, RLS policy) referenced anywhere — §6 CONTEXT, §4 TASK, SQL examples, DoD test queries, §8 IMPORTANT NOTES, and the task-type spec. For each, confirm a corresponding live-query result exists in §6 Live State Verification → Schema verification block. Any entity name without a matching result = STOP and verify before dispatching. The Schema verification gate above is the implementation; this self-check is the enforcement.

** evidence (cc-prompt-skill v1.22 amendment trigger):** Opus shipped the agency-intake-handler CC prompt with two hallucinated DB entities — `aos_workflow_execution_log` (table didn't exist) and a `layer` CHECK constraint (actual constraint name + check_clause differed). Both surfaced as INSERT failures mid-dispatch in CC. The Schema verification row in this gate covers both cases; the gate was not invoked for those assertions. This paragraph adds enforcement by requiring a pre-dispatch sweep against §6 Live State Verification. Memory of "we built this in that session" and doc references (READMEs, prior session notes) are insufficient — tables can be renamed, dropped, or never actually deployed.

---

## 1. CC ACCESS CHECK (every prompt)

```markdown
## CC ACCESS CHECK
Before starting, verify you have access to everything this task requires. If anything is missing, flag it immediately — don't discover it mid-build.

- [ ] {Task-specific access check #1}
- [ ] {Task-specific access check #2}
- [ ] {etc.}
```

Common access checks (include only the ones relevant to the task):
- n8n API key: check via implicit validation — write the migration/task script to `open` the secret and let it raise `FileNotFoundError` if missing. **Do NOT use `py -c "print(os.path.exists(...))"` style checks** — the credential guard fires on any `py -c` command that includes a `.secrets/` path and a `print` call, even for existence-only checks (confirmed). Both Bash `cat`/`test -s` AND `py -c print(exists)` are blocked. Only safe patterns: (a) script-internal `open` that raises on missing, (b) a Python script file (written via Write tool) that reads and uses the secret without printing it.
- Supabase access: `curl -s -X POST .../webhook/{{WEBHOOK_ID}} -d '{"query": "SELECT 1"}'`
- GitHub multi-account ops (mrminor-dev): `gh auth switch --user MrMinor-dev` switches gh CLI context but does NOT update git's credential helper — run `gh auth setup-git` immediately after or all `git push` operations will 403 with old account's cached keyring credentials. See the anti-pattern log. For any DoD that asserts a post-task repository count: use `gh repo list {org} --visibility public --limit 50` at prompt-write time to capture the exact public-repo starting count — default `gh repo list` includes private/archived repos not visible across accounts.
- Supabase safe-sql active (if using safe-sql for verification): `curl -s -H "X-N8N-API-KEY: $KEY".../api/v1/workflows/{{WORKFLOW_ID}}` → confirm `"active": true`. Activate if needed: `POST /activate`.: safe-sql was pre-existing inactive; required diagnostic round before verification could run.
- Supabase service role key: `cat ".secrets/supabase-service-role.txt"`
- File system access: verify you can read the files listed in CONTEXT

The point: CC discovers blockers in minute 1, not minute 30.

**Linux/container paths on Windows:** If a prompt lists `/mnt/skills/user/` or other Linux container paths as a fallback, add an ACCESS CHECK to verify the path is accessible: `ls /mnt/skills/user/ 2>&1`. On Windows, this path does not exist. If inaccessible, proceed with Drive-only files and flag blocked items in output.: prompt assumed container fallback; path was inaccessible on Windows, causing 1/17 skills to be blocked.

**Local script file existence (FIX + script tasks):** If the task targets a `.ps1` or other script at a canonical local path, add a `Test-Path '~/.claude/{script.ps1}'` check. If False: canonical is missing — derive from the Drive backup at `AOS\CC-Config\{script.ps1}`, apply the fix, and create it at the canonical path. The Drive copy becomes stale (expected per §8 pre-existing gap). Note the missing canonical in the feedback file.: `sync-cc-config.ps1` was absent from `~/.claude/`; Drive mirror was the only copy; canonical was created during the fix task.

**New sibling detector dispatched into `cc-config-loop` (or any multi-exemplar tool family):** if the prompt says "no DB/credential access required" AND separately says "match the existing sibling interface," check first whether the existing siblings are a single family or split across families (e.g., DB-backed vs. filesystem-only). "Match the interface" is ambiguous once more than one interface exists among the siblings — state explicitly which family the new detector/tool joins, don't leave CC to infer it from two instructions that can't both be followed literally.: `cc-config-loop`'s 3 existing detectors are all DB-backed (`_emit.py`); a "no DB required" + "sibling to what's there" prompt required CC to ask which one governed rather than silently picking.

---

## 2. RESOURCES (every prompt)

```markdown
## RESOURCES
**Reads:** {list of workflows, files, tables this task will GET/SELECT/read}
**Writes:** {list of workflows, files, tables this task will PUT/POST/DELETE/modify}
```

Every resource that CC will modify goes in WRITES. This enables the COO Resource Conflict Check (pre-write gate) when dispatching parallel tasks.

**Resource naming convention:** Use the most specific identifier.
- Workflows: `n8n:{workflow_id}` (e.g., `n8n:{{WORKFLOW_ID}}`)
- Drive files: full path (e.g., `AOS/Workflows/HAIOS-Intel-3P-Collector/workflow.json`)
- DB tables: `db:{table_name}` (e.g., `db:haios_health_checks`)
- CC config: `cc:{filepath}` (e.g., `cc:~/.claude/CLAUDE.md`)

**Conflict rules (enforced by COO at dispatch, not by CC at runtime):**
- WRITES ∩ WRITES across concurrent tasks → sequence (last-write-wins = data loss)
- READS ∩ WRITES across concurrent tasks → sequence (stale read = wrong decisions)
- READS ∩ READS → safe to parallelize

**Audit prompts whose target calls out to n8n:** If the repo/file under audit POSTs to an n8n webhook, add `n8n API (GET only)` to READS explicitly — CC needs a read-only `GET /api/v1/workflows` to confirm which workflow is actually ACTIVE on that webhook path (Drive READMEs describing webhook paths can be stale; only a live GET is authoritative). Don't leave this as an implicit judgment call for CC to make mid-task.

---

## 3. AP SCAN (every prompt)

**This section is MANDATORY in every CC prompt. It proves the anti-patterns scan happened. An empty or missing AP SCAN = an incomplete prompt.**

COO reads the TASK-TYPE INDEX at the top of `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then), checks each relevant AP, and records the results here. This includes both active APs and graduated APs marked `(G)` — graduated entries live in CC's rules but COO can still spec violations in prompts.

```markdown
## AP SCAN
Task type: {FIX + workflow | BUILD + workflow | TEST | CONFIG EDIT | MIGRATE | AUDIT | CREATE}
Checked: #{N} ({disposition}), #{N} ({disposition}), ...
```

**Dispositions:**
- `embedded` — relevant, warning included in IMPORTANT NOTES or spec
- `N/A` — checked, not applicable to this task
- `⚠️ VIOLATION — {what was changed}` — caught a spec error during scan, fixed before writing

**Example:**
```markdown
## AP SCAN
Task type: FIX + workflow
Checked: #35 (schema verify → embedded in ACCESS CHECK), #47 (stale facts → conditional steps used), #61 (node count → runtime derivation), #72(G) (error path direct → N/A, not touching error path), #76 (Postgres $json → N/A, no downstream DB nodes)
```

**Why this exists:** COO skipped the anti-patterns scan and specced a Code node between Error Trigger and Log HC Error — directly violating (4 reverts across). The gate existed but was invisible. This section makes the scan auditable.

---

## 4. TASK (every prompt)

```markdown
## TASK
{One paragraph: what to do, what the deliverable is, where the result lives.}
```

Keep it tight. If you need more than a paragraph, the details belong in the task-type section.

**Mandatory subsections (include when the trigger condition is met):**

### §EXECUTION BUDGET (REQUIRED if any Schedule trigger node)
- **Cadence:** {every N min/hr/day}
- **Monthly cost:** {N executions / month} (per `HAIOS/Post-Mortems-credential-outage-cascade.md` §6 reference table)
- **Tier:** Standard (2% / ~2.4-hr cadence) | Elevated (5% / ~1-hr) | Critical (10% / ~30-min, requires Jordan sign-off) | Reserved (>10%, architecture review — should route off-platform to CF Worker)
- **Cumulative cap usage after this workflow ships:** {X% of 15,000/mo}
- **Headroom remaining:** {Y executions / Z%}
- **Source:** EXECUTION-BUDGET-LEDGER.md as of {date}
- **Justification (required if Tier > Standard):** {why this workflow needs this cadence}

COO self-check before dispatching: §EXECUTION BUDGET filled with real numbers (not placeholders) → OK. Missing or placeholder values → STOP, fill before dispatching.

### §CREDENTIAL DEPENDENCIES (REQUIRED if any node uses a credential)
- **Credentials touched:** {list from CREDENTIAL-MAP.md by credential_id + name; e.g., `{{CREDENTIAL}}` Postgres}
- **New credential introduced?** {yes/no — if yes, add to CREDENTIAL-MAP.md before dispatching}
- **Companion secrets touched:** {list `.secrets/*` files if any; e.g., `supabase-db-connection.txt`}
- **Canary test post-deploy:** {single command/query that verifies credential works in the new workflow}

### §MONITORING PLATFORM (REQUIRED if workflow's purpose is monitoring/alerting/canary)
- **What does this workflow monitor?** {site / DB / another workflow / execution budget / external service}
- **Does it run on the same platform it monitors?** {yes/no}
- **If yes:** justify why monitor-inside-burning-building is acceptable here. Default: it isn't — move to CF Worker. evidence: all n8n monitors went silent when n8n hit the cap. See `HAIOS/Post-Mortems-credential-outage-cascade.md` §10 AP candidate.

---

## 5. CONTEXT QUERIES (every prompt)

```markdown
## CONTEXT QUERIES
Before Plan Mode, run these semantic searches to gather relevant context:

1. `{domain-specific query}` — {why this matters for the task}
2. `{conventions/patterns query}` — {what conventions to find}
3. `{existing implementation query}` — {what already exists}

Also formulate your own queries based on what you discover. Search results should visibly influence your plan.
```

COO provides 2-3 starter queries tailored to the task domain. CC runs these AND adds its own based on what it learns. The goal: CC enters Plan Mode with institutional context, not just the prompt.

**Note:** When the prompt's CONTEXT section already contains authoritative V3/spec tables, semantic searches are mostly redundant confirmation. Omit CONTEXT QUERIES (or keep 1-2) when the prompt is fully self-contained with authoritative references. Reserve for tasks where CC genuinely needs to discover conventions or find files it doesn't know about.

**Query formulation rules:** 3-8 words, nouns and specific terms, multiple narrow > one broad. See `~/.claude/skills/semantic-search/SKILL.md` for full guidance.

---

## 6. CONTEXT (every prompt)

```markdown
## CONTEXT

### Live State Verification
**MANDATORY for FIX, BUILD-on-existing-workflow, and CONFIG-EDIT-on-workflow-node tasks.** Conditional skip allowed for CREATE / RESEARCH / greenfield BUILD / doc-only tasks — mark the block as `N/A ({one-line reason})` rather than omitting the header, so the gate is visibly checked.

All facts below must come from live queries run at prompt-authoring time. Not from memory, READMEs, prior prompts, or this conversation's earlier context. An empty or placeholder block (or `TBD` values) = an incomplete prompt (same enforcement rule as AP SCAN).

**Workflow identity** — GET `/api/v1/workflows/{id}`, date: {YYYY-MM-DD}:
- id: `{id}` (confirmed name matches: `{workflow name}`)
- version: `{V#}` (from name suffix)
- active: `{true|false}`
- VERIFIED NODE COUNT: `{n}`

**Schema verification** — safe-sql, date: {YYYY-MM-DD}; include only if prompt contains SQL, Supabase REST filters, or DB column references:
- Table `{table}` columns: `{list}`
- CHECK constraints on `{table}`: `{values}`
- Query used: `{one-liner of the safe-sql query}`

**Symptom freshness** — FIX prompts only; one line per symptom with a current LAST VERIFIED timestamp:
- Bug B1 `{one-line symptom}` — LAST VERIFIED: {YYYY-MM-DD} (execution ID: `{id}` / HC: `{row evidence}`)
- Bug B2 `{one-line symptom}` — LAST VERIFIED: {YYYY-MM-DD} (...)

**Node-level verification** — include only if prompt specifies "+N nodes" OR targets a specific node's operation / config:
- Target node `{name}` → operation: `{executeQuery|insert|select|...}` (GET-confirmed)
- ADDED (new nodes): `{list}` | UPDATED IN PLACE (already exist): `{list}`

### General
- **Relevant files:** {list every file CC needs to read, with full paths}
- **n8n instance:** https://{{N8N_HOST}} (if workflow-related)
- **DB:** Supabase at {{SUPABASE_URL}} (if DB-related)
- **Safe SQL webhook:** https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} (if verification needed)
- **Supabase REST API:** Use service role key from `.secrets/supabase-service-role.txt` for write/delete ops. Pattern: `curl -s -X DELETE "{{SUPABASE_URL}}/rest/v1/{table}?{filters}" -H "apikey: $KEY" -H "Authorization: Bearer $KEY"` (if cleanup needed)
- **Repo:** {path} (if code-related)
- **Other endpoints/services:** {as needed}
```

**Why Live State Verification exists:** Audit of that session regressions (`HAIOS/Architecture/SILENT-DROP-REGRESSION-AUDIT.md`) found that share a failure pattern — COO writes prompt facts (workflow ID, node count, node operation type, HC component, bug symptom) from memory or stale docs; CC executes against the wrong target. Silent-drop of user-level rules (bug #21858) weakened the CC-side safety net but the primary cause is COO-authoring. This subsection consolidates into a single structural gate — auditable like AP SCAN. CC reads this block at §1 ACCESS CHECK and flags any divergence between claimed state and live state.

For the General sub-block: only include what's relevant. Don't dump everything.

---

## 7. TASK-TYPE SECTION (load one, customize)

Every prompt contains exactly one task-type section. Identify the task type from the request, then read the matching reference file for the spec block + accumulated session learnings. Copy the spec block into the prompt's §7 and embed relevant session learnings in §8 IMPORTANT NOTES.

| Task Type | Reference File |
|---|---|
| BUILD | `references/task-types/BUILD.md` |
| FIX | `references/task-types/FIX-CORE.md` (spec + Verification Subagent) — COO also reads `FIX-LEARNINGS.md` to select §8 IMPORTANT NOTES entries for the prompt |
| TEST | `references/task-types/TEST.md` |
| DEPLOY | `references/task-types/DEPLOY.md` |
| RESEARCH | `references/task-types/RESEARCH.md` |
| REFACTOR | `references/task-types/REFACTOR.md` |
| MIGRATE | `references/task-types/MIGRATE.md` |
| CREATE | `references/task-types/CREATE.md` |
| CONFIG EDIT | `references/task-types/CONFIG-EDIT.md` |
| AUDIT | `references/task-types/AUDIT.md` |

**Why this is a reference, not an inlined section:** The task-type spec blocks plus accumulated session learnings total ~260 lines. For any given prompt, only one section is relevant. Loading only the relevant task-type file reduces COO's per-prompt read by ~57% with zero content loss. ANTI-PATTERNS.md uses the same TASK-TYPE INDEX pattern — this aligns the two reference files structurally.

---

## 8. IMPORTANT NOTES (every prompt)

```markdown
## KNOWN BUGS (optional — include for FIX/BUILD on existing workflows)
- {Pre-existing bug #1 that isn't this task's target but may interfere}
- {Pre-existing bug #2 — include workaround if known}

## IMPORTANT NOTES
- {Gotcha #1 — things that have broken before on similar tasks}
- {Gotcha #2 — constraints CC must follow}
- {Post-completion warnings — e.g., "MCP access turns OFF after API edits"}
```

Pull from `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` (canonical since then) for relevant warnings.

**Standard gotchas to include when relevant:**
- If task modifies any SSOT doc: "Bump version and updated date in frontmatter."
- If task moves or renames files: "After moving, grep MRMINOR/ for old path and update all cross-references. Exception: skip references where the old path appears as the SUBJECT of a historical log or move description (e.g., 'File X was moved from A to B' or 'N files have stale X path') — changing those corrupts the historical record. Flag them in the build log instead."
- If task appends to any reference file (ANTI-PATTERNS.md, FEEDBACK-LOG.md, skill references): "Before drafting the addition, check if similar content already exists — prefer updating an existing entry over adding a duplicate."
- If task includes conditional edits ("If [text] present, update it"): prefer unconditional phrasing ("Add/update [section] to say...") — conditional language creates ambiguity when the target string has changed or never existed.
- If task installs a hook that fires on events CC cannot trigger (PostCompact, Notification, Stop, PreToolUse): split the verification section into "CC-automated tests" (stdin smoke test, mock JSON test) and "Jordan manual verification" (trigger the real event in a live session). Don't write "run /compact" as a CC verification step — CC can't self-trigger compaction. PostCompact payload fields (summary, hook_event_name) are not in a clean published spec — write hooks to handle missing/empty fields gracefully.
- If task edits Google Drive files with potential CRLF line endings (graduation tasks, doc refactors targeting Drive HAIOS/ or AOS/ files): note 'Use Python script for all edits to this file' in IMPORTANT NOTES upfront. Edit tool cannot match multi-line old_string in CRLF files; discovering this mid-task costs a diagnostic cycle. CRLF convention + Python fix pattern: see CLAUDE.md.
- If task writes a Python script to PUT an n8n workflow: include in IMPORTANT NOTES "Reference `N8N-API-REFERENCE.md` PUT body section for the current confirmed-safe `settings` field whitelist before writing the script."
- If task is a BUILD for a webhook workflow with auth + smoke tests: specify the webhook auth header name and format explicitly in the §SMOKE TESTS section (e.g., `x-intake-secret: {value from secrets-file}`). CC must not need to infer it by inspecting a prior workflow — that round-trip delays smoke test authoring and risks picking the wrong header. Also document E2E bypass headers (e.g., `x-e2e-bypass`) and any IP-injection constraints (Note: CF ignores `X-Forwarded-For`). The whitelist expanded (+ →); a stale list causes 400 errors.: 3 fix iterations, all due to PUT body field mismatches. Reference: `AOS/Domains/Infra/N8N-API-REFERENCE.md` → PUT body section.

---

## 9. EXECUTION POLICY (every prompt)

```markdown
## EXECUTION POLICY
**Stop criteria:** {concrete condition — "stop when tests pass", "stop when HC returns ok", "stop when all target files exist and diff clean against spec"}
**Fallback pattern:** {unhappy path — "if {resource} not found, {action} — do not guess"}
**Verification method:** {run service / browser test / post-run DB query / safe-sql check / manual review}
```

Anthropic's agentic-workflow non-negotiables. Ambiguous stop = overthinking. Missing fallback = hallucinated output. Unspecified verification = untested work.

**Required when:**
- **Stop criteria** — every prompt. Default `stop when DoD all checked` is fine for bounded tasks; agentic/open-ended tasks need explicit concrete conditions.
- **Fallback pattern** — every prompt that reads external resources (files, DB, APIs). Omit only for pure-authoring tasks with no reads. Template: "If {read target} returns {bad state}, {specific action} — do not infer/guess/self-heal."
- **Verification method** — mandatory for BUILD, FIX, DEPLOY, MIGRATE. Recommended for TEST, REFACTOR, AUDIT. Usually omitted for CREATE, CONFIG-EDIT, RESEARCH (where the output IS the deliverable). Workflow tasks: reference `.claude\rules\workflow-verification.md` 4-layer standard (project-level since then; `~/.claude/rules/workflow-verification.md` is a retired husk).

**Compaction trigger (mandatory for tasks >20 EXECUTE rows or >5 file reads):**
At natural breakpoints (after each edit group, after pre-PUT verification):
```
/compact keep: [routing table or DoD checklist, completed items, current
edit in progress]. drop: [file contents already processed, tool outputs
from completed steps]
```
Do NOT wait for autocompact. Compact before context fills, not after. Context rot zone on 1M window is ~300-400k tokens (see `CC-DEVELOPMENT-PLAYBOOK.md` § Context Rot -- Quantified).

---

## 10. DEFINITION OF DONE (every prompt)

```markdown
## DEFINITION OF DONE
- [ ] {Primary deliverable}
- [ ] {Verification step}
- [ ] {Doc update if needed}
- [ ] {Cleanup if needed}
- [ ] Post-activation closeout: workflow.json mirror refreshed + snapshots/{ts}-post-{verb}.json saved (per ~/.claude/rules/n8n-rollback.md § Post-Activation Persistence). [Include only for workflow BUILD/FIX prompts]
- [ ] README review completed (stale cadence, changed node count, new error-handling paths). [Include only for workflow BUILD/FIX prompts — global CLAUDE.md already requires this on every workflow PUT; stating it explicitly in the DoD stops it being caught only by cross-referencing the global rule]
- [ ] Feedback file written per standard protocol (see global CLAUDE.md)
```
