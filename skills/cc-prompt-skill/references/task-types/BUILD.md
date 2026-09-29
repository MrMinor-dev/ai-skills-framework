# BUILD Task Type

Workflows, features, components.

**Recommended effort:** `xhigh`

CC default. Not typically lowered. Override to `max` only for genuinely hard problems (use deliberately — diminishing returns).

<!-- COO uses this file when writing a CC prompt for a BUILD task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## SPEC
{Full specification. For workflows: node-by-node. For features: component breakdown.
Include: inputs, outputs, connections, error handling, data flow.}

## TEST PAYLOADS + EXPECTED RESULTS
{Table so CC can assert correctness programmatically, not infer from side effects.
Each test includes Known Limitations — what it CAN and CANNOT verify.}
| Payload ID | Key Input | Expected Output | Verification Query | Known Limitations |
|---|---|---|---|---|
| test-001 | {input} | category: X | SELECT ... WHERE .. | {what this test CANNOT verify and why} |

{Mark untestable tests explicitly -- don't list tests that can't actually run.
If a test trigger differs from production trigger (e.g., webhook vs Gmail Trigger),
document the payload structure difference and what that means for test accuracy.
IMPORTANT: Verify all payload field names match the workflow's actual validation/input node logic, not the natural English field names. GET the workflow and inspect the input validation node before writing the payload -- field names like `expense_date` vs `date` are not detectable from context alone. A wrong field name causes a validation pass-then-INSERT-fail that may trigger Error: Log unexpectedly.}
{For HC verification queries: source the `component` filter value from the workflow's actual HC node SQL (GET workflow -> inspect Success: Log `parameters.query`). Never use a component name from memory. }
{DoD test queries: verify all table names used in DoD test payloads exist in the live DB before writing the prompt -- run `SELECT 1 FROM {table} LIMIT 0` via safe-sql or check `SELECT table_name FROM information_schema.tables WHERE table_name LIKE '%{keyword}%'`. Stale table names silently prove nothing -- behavior is "verified" against a 404, not the real data path. applies. }
{Pre-build inspection queries: always include `SELECT COUNT(*) FROM {table} WHERE {condition}` in addition to `LIMIT N` row samples. A LIMIT query creates the impression there are only N rows; the real count may be much higher.: pre-build showed 5 rows; execution processed 15. COUNT(*) would have flagged the discrepancy.}
{HC node SQL spec: always specify `business_id: 'mrminor'` explicitly. Never use `'aos'` -- that is an invalid business value in production. Recurring pattern: specs use 'aos' as shorthand; SSOT is HEALTH-CHECK-PATTERNS.md which specifies 'mrminor'. }
{SQL field inclusion for LLM-processed content: when the workflow aggregates DB fields and passes them to an LLM (e.g., Wikifier, Claude API), explicitly specify WHICH fields to include in the aggregation SQL and WHY. Do NOT include all available fields by default. Raw JSONB fields (e.g., `action_items`) are unreadable in LLM output and inflate token usage. Specify the target fields and exclude operational/raw data that doesn't belong in the output (e.g., wiki pages, digest, report).: action_items included in SQL aggregate -> unreadable in wiki + inflated Haiku extraction output. Fix: landscape_summary only.}

## AUDIT CRITERIA
{Embed the relevant quality checklist. For workflows: 65-point audit criteria.
For code: testing requirements. CC builds to pass from the start.}
```

## Pre-Build Gate — External Integrations (v4.14 addition)

For any BUILD touching external APIs, databases, or third-party data sources, add the following to IMPORTANT NOTES in the CC prompt:

```markdown
**Phase 0 — Live Recon (required before any integration code):**
For each external data source in scope, run a throwaway script that:
1. Hits the API/DB with real credentials
2. Dumps a sample response to `docs/recon/{source}-sample.json`
3. Exits cleanly
Do NOT write integration code until recon confirms actual field names, auth flow, and response shape.
```

Also add to DEFINITION OF DONE for multi-task builds:
```markdown
- [ ] Each completed task committed with message `[task-complete] {task-description}` for idempotent re-runs
```

Source: Mike Reilly Pulse primer, IS4AI Day 1. Full pattern: Playbook v4.14 §Research Before Plan + §Verification Loops.

## Session Learnings

- **BUILD/FIX prompts that add or modify a SQL INSERT/UPDATE built as a concatenated string in a Code node:** provide the exact target column-list + VALUES template **verbatim** in the SPEC — do NOT describe it as "map field X into the INSERT." Hand-built SQL string concatenation drops delimiters silently (Note: adding `signal_type`/`actionability` to the analyzer's `Parse Response` grouped insertSql dropped the closing quote on `signal_source` → best_practice-path SQL syntax error, caught only by COO code-review of the live node, not by the build's own manual run). Require a DoD unit test that builds the generated SQL for a sample row and asserts quote-balance, including apostrophe-containing values.

**Workflow build references:**
- **Standing n8n conventions:** `.claude\rules\n8n-workflows-core.md` + `.claude\rules\n8n-workflows-learnings.md` + `.claude\rules\workflow-verification.md` (all project-level since then; user-level copies never existed / are retired husks). Do NOT repeat rule content in prompts. In IMPORTANT NOTES write: "Standard n8n conventions apply (see `n8n-workflows-core.md` + `n8n-workflows-learnings.md`). 4-layer verification required (see `workflow-verification.md`)."
- **API reference:** `AOS/Domains/Infra/N8N-API-REFERENCE.md`
- **Prompt-specific (not in rules — include when relevant):** Separate unit vs integration tests. Session number in prompt header. Assertion patterns (count modified items). HTTP nodes to internal webhooks use `onError: continueRegularOutput` (N8N-API-REFERENCE pitfall #18).
- Section insertion in existing Code nodes: name adjacent sections explicitly — "after [Section X], before [Section Y]". Positional phrases like "before any closing/footer" are ambiguous when the last section has presentation significance (e.g., HAIOS Messages is last by design). see the session notes quarantine section placement.
- Inserting literal text into Code node JS template literals: when the SPEC section includes exact text to be inserted into a JavaScript template literal (backtick string), clarify whether `{placeholder}` notation is literal markdown (kept as-is in the generated output) or should be interpolated as `${ctx.field}` (evaluated at runtime). Inspect the surrounding template literal for existing interpolation patterns and match them.: spec used `{workflowId}` which could have been literal or interpolated -- CC matched existing `${ctx.workflow_id}` style correctly but this was a judgment call, not explicit.
- Targeted replacement in Code nodes: provide a unique string anchor (not a line number) for the exact block to change. Line numbers shift as code evolves; unique strings like `workflow_name: 'HAIOS-Infra-DB-Security',\n detection_type:` are stable and unambiguous. Always specify "do NOT change occurrences outside this block" when the same field/value appears in multiple branches. Assert `occurrences == 1` in the replacement script as a safety gate.: unique anchor enabled zero-risk surgical replace across 4 detection_type occurrences in one Code node.
- **Error handling (MANDATORY for every n8n workflow BUILD):** Every workflow must have an Error Trigger node per `.claude\rules\n8n-workflows-core.md` § Error Handling (MANDATORY). The BUILD prompt §7 TASK section MUST declare `interactive: true|false`. When `interactive: true`, must also specify `interactive_slack_channel: <C-ID>` (e.g., `{{SLACK_ID}}` for `#blog-interview`). DoD verification: GET workflow, confirm Error Trigger node exists, confirm correct topology — operational = direct `Error Trigger → Error: Log`; interactive = parallel fan-out `Error Trigger → [Error: Log, Slack alert]`. Default if undeclared = operational (DB-only); CC will NOT add Slack on its own. Gap closed — BLG-03 and BLG-07 originally built without Error Triggers because the prior rules described HOW but not WHETHER.

- **Workflow registration (MANDATORY for every n8n workflow BUILD):** Every new n8n workflow must be registered in `haios_workflow_registry` as a DoD step. Without a registry entry the remediation trigger uses a flat 24h liveness window — correct for daily workflows but causes false-positive silent failure alerts for weekly/monthly cadences (Note: Doc Governance false-alarmed Mon–Sat weekly). Add a DoD checkbox: "INSERT into `haios_workflow_registry` with correct `liveness_window_hours`."
  Live columns (+ verified): `workflow_id, name, layer, category, is_killable, description, created_at, liveness_window_hours`.
  Set `liveness_window_hours` per cadence:
| Run cadence | liveness_window_hours |
|---|---|
| Hourly | 3 |
| Every N hours | N × 2 |
| Daily | 24 (explicit — NULL would *suppress* monitoring, not default to 24h) |
| Weekly | 192 (8 days) |
| Monthly | 800 (~33 days) |
| On-demand / no schedule | NULL (registry suppression — observing-caller taxonomy, `HEALTH-CHECK-PATTERNS.md` §On-Demand). Do NOT add the component to the hardcoded `PG: Find Silent Failures` NOT-IN list — it's frozen/legacy; registry-NULL is the mechanism. |
  `layer` must be lowercase (`haios`, `aos`). `is_killable = false` unless workflow explicitly supports mid-run termination.

- SQL INSERT to existing tables: always include the exact column list — never assume column names. Verify against `SELECT column_name FROM information_schema.columns WHERE table_name = '{table}'` via safe-sql if uncertain. For `haios_workflow_registry` INSERT: live columns are `workflow_id, name, layer, category, is_killable, description, created_at, liveness_window_hours` (added liveness_window_hours confirmed live schema). Do NOT infer columns from the prompt context — query `information_schema.columns` before writing the INSERT.
- **Schema SSOT is now the live DB (archived). Do NOT reference it.** The live DB is the SSOT. Schema verification = `information_schema` queries via safe-sql.
- **HTTP Request node credential tables must include `nodeCredentialType`.** When a BUILD prompt lists credentials for HTTP Request nodes, the credential table must include a `nodeCredentialType` column (e.g., `googleDriveOAuth2Api`, `anthropicApi`, `httpHeaderAuth`). Without it, CC must inspect an existing workflow to determine the correct type — wasted round-trip.: Anthropic credential listed name + ID but not type; CC found correct type (`anthropicApi`) by inspecting Wikifier V2. Format: `| Service | Credential Name | Credential ID | nodeCredentialType |`.
- REST API PATCH/POST bodies (Supabase or other): same rule applies. Verify all column names via `information_schema.columns` query before writing the body. A column referenced in a PATCH that does not exist will 400 or silently no-op. See ANTI-PATTERNS #35.
- Batch workflow modification prompts (multiple workflows, same operation): when any target workflow has multiple terminal branches, specify the exact terminal node name in the target table — never leave CC to infer "the main branch." Add a column `Terminal Node` to the target table (e.g., `| WF ID | Name | Terminal Node | Drive Folder |`). Without this, CC must make a judgment call that may not match COO intent.: Slack Inbound had 5 terminal nodes; prompt said "pick the main one" — CC chose correctly but the ambiguity is avoidable.
- Batch workflow target tables for health check / monitoring upgrades: add a `MCP-enabled?` column so CC knows which PUT responses to flag vs. accept. For workflows that have never had MCP access (chat trigger utilities, non-MCP schedules), set `MCP-enabled? = N/A` — MCP=False after PUT is expected and correct. For MCP workflows, set `MCP-enabled? = Yes` — MCP=False after PUT should be flagged. Without this column, CC must guess whether MCP=False is a bug or expected behavior.: Gmail Query MCP=False was correct but caused ambiguity in the PUT verification step.
- Rollback snapshot naming: do NOT re-specify the snapshot filename format in prompts. The canonical naming convention is in `.claude\rules\n8n-rollback.md` (`YYYYMMDD-HHMMSS-pre-{verb}.json`) -- project-level since the CC-RULES-CONSOLIDATION-SPEC fold; the user-level copy is a retired redirect stub. If a prompt specifies a different filename (e.g., `rollback-pre-cron.json`), CC will use the rule convention, creating a discrepancy between what the prompt promised and what was saved. Reference the rule instead: "Save rollback snapshot per `n8n-rollback.md` naming convention.": prompt specified custom filename; CC used rule convention.
- **BUILD prompts with iterative PUTs must reference `.claude\rules\n8n-rollback.md` in IMPORTANT NOTES.** The snapshot rule is a CC rule, but CC is unlikely to follow it during a fast-moving iterative BUILD unless the prompt explicitly says "snapshot before every PUT per n8n-rollback.md.": 4 iterative PUTs during wikifier BUILD — snapshot protocol not followed because prompt didn't mention it.
- **Schema spec tables in BUILD prompts must include CHECK constraint allowed values.** Include a column for constraints: `| column | type | constraints |` with values like `model_used | text | CHECK ('haiku','sonnet')`. Omitting allowed values forces CC to discover them from test failures (adds 1+ fix cycles per undocumented constraint).: model_used CHECK constraint omitted from spec — discovered at runtime.
- **Claude API JSON fence stripping.** Any prompt that includes a Code node parsing Claude API response as JSON must note: "Claude may return JSON wrapped in ```json...``` fences even with a JSON-only instruction. The parse node must strip fences before JSON.parse." Use regex pattern.: Claude fenced JSON caused JSON.parse failure in Parse Wikifier Response.
- **Compile Report / Code node filter for new check with different field names:** When a new parallel check produces items with different field names than existing checks (no shared `check_id` or equivalent), the filter approach for that check's items must be explicitly specified in the spec. Use field-presence: `items.filter(item => item.json.unique_field !== undefined)` where `unique_field` is a field only present in that check's output. Never specify `item.json.$node === 'Node Name'` -- `$node` is an n8n expression context variable, not a JSON data property; it returns `undefined` when accessed as `item.json.$node`, silently producing an empty filter result..: spec suggested `$node` filter; correct filter was `item.json.check_type !== undefined`.
- Snapshot count in verification criteria: when a FIX prompt includes a "Prune snapshots" step, the rollback snapshot taken in Step 1 adds 1 file before pruning runs. Verification criteria saying "snapshots/ contains exactly N files" must account for this. Write the expected count as N+1 (3 kept + 1 new = 4) or add a note: "N kept from list + today's pre-fix snapshot = N+1 total.": verification said "exactly 3" but correct final count was 4.
- FOLDER CLEANUP "MUST contain EXACTLY" language: when a FIX prompt specifies that the workflow folder must contain exactly certain files, this applies to workflow artifacts only. Session files -- CC-BUILD-LOG-*.md, CC-FEEDBACK-*.md, CC-FEEDBACK-PROCESSED-*.md -- remain in the folder per global CLAUDE.md ("Leave feedback and build log in place for COO review"). Do not include session files in the EXACT count. The CC-PROMPT-*.md file is archived to Archive/CC-Prompts/ per the standard protocol, so it should not appear in the target file list either.: prompt listed 3 target files + snapshots/; actual final state correctly had +many sessions files (build log + feedback) per global CLAUDE.md hierarchy.
- **Drive API v3 constraints:** Any BUILD prompt using Google Drive file search must include this callout in IMPORTANT NOTES: "`in ancestors` and `not in ancestors` are NOT supported by Drive API v3 -- both return HTTP 400. Use chained `in parents` queries instead. Pattern: Code node builds `('id1' in parents or 'id2' in parents) and trashed=false`, fed as `={{ $json.q }}` to next HTTP Request. Handle 0-dir edge case with dummy query `name='__NO_DIRS__' and trashed=false`. See `.claude\rules\n8n-workflows-core.md` External API Constraints and `N8N-API-REFERENCE.md` pitfall #40."
- **Context-setter Code node vs Set node:** When a node must output exactly 1 item for downstream cross-node `.first` references, specify Code node with `runOnceForAllItems: true`, NOT a Set node. A Set node with 2+ inputs from a Merge passes through all items; a Code node with `runOnceForAllItems` outputs exactly 1. Specify explicitly in the spec: "Set: Context — Code node, mode: runOnceForAllItems."
- **Post-activation closeout (MANDATORY for every workflow BUILD/FIX):** After verifying the PUT is active, persist two artifacts in the workflow folder: (a) overwrite `workflow.json` with a fresh GET (always-current SSOT mirror), and (b) save `snapshots/{ts}-post-{verb}.json` (preserves each built state so a later edit cannot orphan it). Both are required per `.claude\rules\n8n-rollback.md` § Post-Activation Persistence. Add as a DoD checkbox: "Post-activation closeout: workflow.json mirror refreshed + snapshots/{ts}-post-{verb}.json saved.": orphaned `6fd9ba0a` chunking work because no post-build persistence was in place.

- **Schedule-triggered workflow verification is Jordan-dependent:** a BUILD on a schedule-triggered n8n workflow can't be self-verified by CC (`POST /run` → 405 + unreliable n8n MCP auth). Plan the Jordan-fired manual-run path at authoring time — see `FIX-CORE.md` § Schedule-Triggered Verification. Anchor `/goal` to "built + PUT structurally verified"; L1/L2 gated on the Jordan-fired or next-scheduled run.
