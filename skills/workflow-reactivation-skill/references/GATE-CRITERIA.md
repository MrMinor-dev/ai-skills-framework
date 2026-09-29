---
title: Workflow Reactivation — Gate Criteria
layer: AOS
updated: 2026-06-01
<!-- Changelog: — §L2 reactivation-boundary canary (authenticated read per distinct credential, presume-stale, NOT a health probe) + §L6 credential-canary-of-record row; matches SKILL.md v1.7 (C4 of that session-lineage post-mortem). | — added §L3 HC field-value check (layer/business_id correctness); Code node mode row already present. -->
---

# Gate Criteria (Detailed)

Detailed pass/fail tests for each gate. Load on demand during per-workflow inspection.

---

## §L1 — Ledger + Budget

Source: `EXECUTION-BUDGET-LEDGER.md`

| Check | Pass | Fail → Action |
|---|---|---|
| Row exists in ledger | Workflow ID found in scheduled or webhook table | Missing → add row before proceeding |
| Cadence confirmed | No TBD-CC in cadence column | TBD-CC present → cadence verification required before activating |
| HEALTH column | HEALTHY or STALE | DEGRADED → hard block, log HOLD-L1 |
| Cap math | Running total + this workflow ≤ 11,250/mo | Over 75% → escalate to Jordan before activating |

**Cap budget is cumulative.** Run waves in order so high-cost workflows (Intake Processor 2,880/mo, Error Alerter 2,880/mo) are added last. If adding a workflow would cross 11,250, pause and get Jordan decision.

---

## §L2 — Credential Verification

Source: `CREDENTIAL-MAP.md §1`. Cross-reference workflow JSON from `n8n:get_workflow_details`.

**Step 1:** Identify every credentialed node in the workflow (Postgres, Gmail, Slack, Google Drive, HTTP with auth, Anthropic, n8n API, etc.).
**Step 2:** For each credential name, find the row in CREDENTIAL-MAP.md §1.

| Check | Pass | Fail → Action |
|---|---|---|
| Credential in map | Row found in §1 | Missing → add row; flag for rotation audit |
| CREDENTIAL-MAP HEALTH | HEALTHY or STALE | DEGRADED → hard block, log HOLD-L2 |
| Boundary canary (mandatory — lineage) | A real **authenticated read** for each *distinct* credential returns data (safe-sql `SELECT 1` for Postgres; an authenticated GET that returns data for an API key) | Fails, OR only a connectivity/health probe was run → treat as DEGRADED, log HOLD-L2. A green `connected`/`healthy` probe is NOT a pass — it can sit over a dead key (Note: `n8n_list_workflows` 401'd while the diagnostic reported connected). |
| Rotation age | Last rotated ≤ 90d | > 90d → flag; reactivate only if spot-check passes |

**Boundary-canary method (lineage — presume stale until proven).** A credential behind a *dormant* workflow is presumed STALE: nothing executed between deactivation and now to surface drift (Postgres + n8n-API both detonated on the first post-reactivation exec). Run a real authenticated read per *distinct* credential, once per boundary (not once per workflow), and log the result in REACTIVATION-LOG. Methods: Postgres → safe-sql `SELECT 1`; n8n API → an authenticated `GET /workflows` that returns data (NOT a `health_check`/`diagnostic` probe — those report `connected` over a 401); Google Drive/Gmail OAuth2 → n8n UI must not show a refresh-token error; other API keys → one authenticated read against a real endpoint. Last-rotation date in CREDENTIAL-MAP is supporting evidence, not a substitute for the live read.

---

## §L3 — Structural Hygiene

Source: workflow JSON from `n8n:get_workflow_details`. Inspect `nodes[]` array.

### Error handling wiring

| Check | Pass | Fail → Action |
|---|---|---|
| Error Trigger node present | Node with `type: n8n-nodes-base.errorTrigger` exists | Missing → dispatch CC to add |
| Error Trigger connected | errorTrigger leads to Postgres INSERT or Slack alert | Dangling (unconnected) → dispatch CC |
| Error → haios_health_checks | Postgres INSERT with `state='error'` on error path | Missing → dispatch CC |
| Success → haios_health_checks | Postgres INSERT with `state='ok'` on main completion path | Missing → dispatch CC |

### Node typeVersions

Minimum acceptable versions (anything below = dispatch CC for bump):

| Node type | Minimum typeVersion |
|---|---|
| `n8n-nodes-base.postgres` | 2.5 (2.4 acceptable if no known issues) |
| `n8n-nodes-base.httpRequest` | 4.2 |
| `n8n-nodes-base.scheduleTrigger` | 1.1 |
| `n8n-nodes-base.slack` | 2.3 |
| `n8n-nodes-base.gmail` | 2.0 |
| `n8n-nodes-base.googleDrive` | 3.0 |
| `n8n-nodes-base.if` | 2.0 |
| `n8n-nodes-base.merge` | 3.0 |

Reference for any known platform-specific version issues beyond this table.

### Other checks

| Check | Pass | Fail → Action |
|---|---|---|
| Webhook paths | Production webhook URL — no `-test` substring in path | Test path in prod config → dispatch CC |
| Workflow description | `workflow.description` field non-empty | Empty → dispatch CC to populate from README or workflow purpose |
| availableInMCP flag | `settings.availableInMCP: true` (for workflows COO should be able to inspect) | Missing flag → dispatch CC to enable |

### Code node configuration

| Check | Pass | Fail → Action |
|---|---|---|
| Code node mode | Every `n8n-nodes-base.code` node has `"mode": "runOnceForAllItems"` in `parameters` | Missing → dispatch CC to add `"mode": "runOnceForAllItems"` to each affected node's `parameters` object. Two failure modes if omitted: (a) JsTaskRunnerSandbox realm isolation crash; (b) silent wrong output when node uses `$input.all` with multiple items — runs once per item instead of once for all, e.g. N Slack posts instead of 1 digest. Neither failure is visible without live testing. |

### HC field values

Applies to any workflow with a `haios_health_checks` INSERT node. Check the Postgres INSERT parameters in the workflow JSON.

| Check | Pass | Fail → Action |
|---|---|---|
| HC `layer` value | `layer` is one of `HAIOS`, `AOS`, `Agency` (uppercase, no maturity suffix) | Wrong (e.g. `'L2'`, `'aos'`, `'infra'`) → dispatch CC to correct. Root cause of that session Weekly Audit bug. See `HEALTH-CHECK-PATTERNS.md` for canonical INSERT. |
| HC `business_id` value | `business_id` is `'mrminor'` | Wrong or missing → dispatch CC to set `'mrminor'`. |

---

## §L4 — README + Snapshot

Source: Google Drive (search by workflow name in layer folder).

### Finding the workflow folder

Layer is determined by workflow name prefix (see WORKFLOW-LAYER-MAP.md):
- `Agency:` / `AGENCY-` prefix → search `AGENCY/Workflows/`
- `AOS:` / `[AOS]:` / `HAIOS:` prefix → search `AOS/Workflows/`

Use Google Drive search: `name contains '{workflow-keyword}' and '{layer_folder_id}' in parents and mimeType = 'application/vnd.google-apps.folder'`

Drive folder IDs: AGENCY = `{{DRIVE_ID}}` | AOS = `{{DRIVE_ID}}`

### README checks

| Check | Pass | Fail → Action |
|---|---|---|
| README.md exists | File found in workflow folder | Missing → dispatch CC to create from workflow JSON + purpose |
| README freshness | `updated:` frontmatter date ≤ 90d ago, OR content matches current trigger/credential/schema architecture | Stale relative to last significant change → dispatch CC to update |

**README staleness judgment:** Age alone is not a fail if the workflow hasn't changed. Staleness = README describes a different trigger type, different credentials, or missing nodes that now exist. When in doubt, compare README §Architecture section to current workflow JSON.

### Snapshot checks

| Check | Pass | Fail → Action |
|---|---|---|
| snapshots/ folder exists | Subfolder named `snapshots` found in workflow folder | Missing → dispatch CC to create folder + export |
| Snapshot file exists | At least one `.json` file in `snapshots/` | Missing → dispatch CC to export current workflow JSON |
| Snapshot is current | Most recent snapshot filename contains current `versionId` OR snapshot JSON `versionId` field matches n8n | Stale → dispatch CC to re-export |

**versionId source:** `workflow.versionId` from `n8n:get_workflow_details` response.

**Snapshot naming convention:** `{workflow-name}-{versionId}.json` or `{workflow-name}-{YYYY-MM-DD}.json`. Either is acceptable.

---

## §L5 — Architecture + Remediation

Source: `haios_remediation_log` (via safe-sql or direct DB query when safe-sql is live).

### Remediation log check

Query: `SELECT * FROM haios_remediation_log WHERE workflow_name ILIKE '%{workflow-keyword}%' AND status IN ('pending', 'in_progress')`

| Check | Pass | Fail → Action |
|---|---|---|
| No open entries | 0 rows returned | Any pending/in_progress → escalate to Jordan, document decision in REACTIVATION-LOG |

**Jordan decision options for open entries:**
1. Resolve the remediation before activating (preferred)
2. Accept the risk and activate with documented rationale
3. Defer activation to post-June-1

### Architecture review

| Check | Pass | Fail → Action |
|---|---|---|
| Not superseded | No newer workflow version or CF Worker replacement | Do not activate, update ledger |
| Trigger type fit | Current trigger is appropriate for architecture | Flag as POST-JUNE-1 migration candidate in tracker (not a hard block unless Jordan decides) |

**Trigger type assessment:**
- Polling schedules checking for new data every 15min → candidate for webhook-event-triggered (Phase B)
- Gmail/Drive triggers polling every minute → event-driven but costs only on actual events (confirm observed rate is acceptable)
- Any schedule > 1,440/mo → confirm Jordan-approved in ledger Notes column

---

## §L6 — Post-Reactivation Verification

### Activation

1. Activate workflow in n8n (toggle active = true)
2. For **Schedule triggers** with cadence > 5min: manually trigger one execution via n8n UI or MCP
3. For **Webhook triggers**: send one test POST payload
4. For **Gmail/Drive/Slack triggers**: wait for one natural event OR send a test event

### Verification checks

| Check | Pass | Fail → Action |
|---|---|---|
| Execution status | n8n execution shows `succeeded` | `error` or `crashed` → deactivate immediately |
| Credential canary of record | First post-activation execution authenticates cleanly | A credential-auth failure here (as opposed to a logic error) means the L2 boundary canary was skipped or the credential went stale between canary and activation → deactivate, re-run the L2 authenticated read, then retry |
| haios_health_checks row | Row with `state='ok'`, `workflow_id='{id}'` appears within 2 min | Missing → HC wiring failure; deactivate, re-run L3 |
| No error rows | No `state='error'` rows for this component in last 5 min | Error rows → deactivate, diagnose |
| Slack notification | Alert fires in expected channel (if wired) | Missing → note in tracker; not a hard fail if workflow has no Slack output |

### Rollback

If any L6 check fails:
1. Deactivate workflow in n8n immediately
2. Insert row into `haios_remediation_log` with detection_type = 'reactivation_failure'
3. Log HOLD-L6 in REACTIVATION-LOG with error details
4. Do not retry without diagnosing root cause
