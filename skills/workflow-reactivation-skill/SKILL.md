---
name: workflow-reactivation-skill
description: "Per-workflow checklist for moving an n8n workflow OUT of or back INTO production. Use when: reactivating a workflow after deactivation, putting a workflow back into production, June 1 reactivation; ALSO when parking, deactivating, decommissioning, or marking a workflow Not-In-Prod — parking requires the §2.5 haios_workflow_registry suppression row or the silent_failure monitor false-fires."
version: 1.8
created: 2026-05-26
updated: 2026-06-17
<!-- Changelog: — description expanded to ALSO trigger on parking/deactivation/Not-In-Prod (was reactivation-only), so §2.5 registry suppression fires at park time. Root cause of the silent_failure false positive — 3 Blog workflows parked without §2.5 registry rows because the skill never triggered on parking. Added Quick-Reference both-directions note + §2.5 trigger banner. | — L2 reactivation-boundary credential canary (authenticated read, presume-stale default, NOT a health probe) + L6 first-exec-is-canary framing. Closes criterion C4 of the-lineage post-mortem. | — added L3 HC field-value check (layer/business_id) to SKILL.md + GATE-CRITERIA.md -->
layer: AOS
---

# Workflow Reactivation Skill

Six hard-gate checklist run per workflow before touching the active toggle. ALL gates must pass — no note-and-proceed. Designed for the June 1 mass reactivation; reusable for any return-to-production event.

## Quick Reference

This skill covers BOTH directions. **Reactivation / return-to-prod:** run gates L1–L6 (§2). **Parking / deactivation / Not-In-Prod:** run §2.5 — the mandatory step is the `haios_workflow_registry` suppression row; skip it and the silent_failure monitor false-fires within 7 days (evidence: 3 Blog workflows parked without it).

Gate order matters: L1 blocks on budget/health, L2 on credentials, L3 on structural hygiene, L4 on docs/snapshots, L5 on architecture/remediation, L6 post-activation verification. First fail stops that workflow — log blocker, move to next. Activate in wave order (WORKFLOW-LAYER-MAP.md) so dependencies precede dependents and cap overruns surface early.

## Workflow

### 1. LOAD CONTEXT

```
Filesystem:read_file → AOS\Domains\Infra\EXECUTION-BUDGET-LEDGER.md
Filesystem:read_file → AOS\Domains\Infra\CREDENTIAL-MAP.md
```

Create session tracker: `AOS\Domains\Infra\REACTIVATION-LOG-{YYYY-MM-DD}.md`

Tracker columns: `| Workflow ID | Name | Wave | L1 | L2 | L3 | L4 | L5 | L6 | Status | Blocker |`
Status values: `ACTIVE` / `HOLD-{gate}` / `PENDING-CC` / `SKIP-ARCHIVED`

Reference `references/WORKFLOW-LAYER-MAP.md` for layer paths and wave order.

### 2. RUN GATES (per workflow, in wave order)

Full pass/fail criteria: `references/GATE-CRITERIA.md`

**L1 — Ledger + Budget**
- Row in EXECUTION-BUDGET-LEDGER.md with confirmed cadence (no TBD-CC)
- HEALTH ≠ DEGRADED
- Running cap total + this workflow ≤ 11,250/mo (75% of 15k hard cap)

**L2 — Credential verification**
- Every credential used by this workflow → row in CREDENTIAL-MAP.md §1
- No DEGRADED credentials
- **Reactivation-boundary canary (mandatory — lineage).** A credential backing a *dormant* workflow is presumed STALE until proven otherwise — nothing executed between deactivation and now to surface drift (Postgres + n8n-API both detonated on the first post-reactivation exec). Before activating, run a real **authenticated read** for each distinct credential this workflow uses (e.g., safe-sql `SELECT 1` for Postgres; an authenticated GET that returns data for an API key). A connectivity/health probe is NOT sufficient — `connected: true` / `healthy` can sit over a dead credential (false-healthy: `n8n_list_workflows` 401'd while the diagnostic reported connected). Canary each *distinct* credential once per boundary (not once per workflow); log the per-credential result in the REACTIVATION-LOG.

**L3 — Structural hygiene**
- Error Trigger node present and wired to haios_health_checks INSERT or Slack alert
- haios_health_checks INSERT on success path (state='ok')
- haios_health_checks INSERT on error path (state='error')
- No deprecated typeVersions (ref + GATE-CRITERIA.md §L3 version floor table)
- Every `n8n-nodes-base.code` node has `"mode": "runOnceForAllItems"` in parameters (crash + silent wrong output)
- `haios_health_checks` INSERT field values: `layer` = workflow name-Layer (`HAIOS`/`AOS`/`Agency`), `business_id` = `'mrminor'`
- Webhook nodes: production path only (no `-test` in path)
- Workflow description field populated
- Workflow name format: `[Layer]: [Domain] - [Workflow Name] [Lx]` — no `Vx` version suffix
- Drive folder name = sanitized n8n name (`:` → `-`); **never the workflow ID**
- `WORKFLOW-LAYER-MAP.md` has an entry for this workflow ID → n8n name + Drive folder path

**L4 — README + snapshot**
- README.md exists in workflow's Drive folder (layer from WORKFLOW-LAYER-MAP.md)
- README reflects current architecture — not stale relative to last significant workflow change
- JSON snapshot exists in `snapshots/` subfolder
- Snapshot versionId matches current n8n workflow `versionId`
- **Folder hygiene:** Scan for stale support files (session analysis docs, old auto-prompts, pre-fix notes, resolved incident docs). For each file found:
  - If status is RESOLVED / superseded / pre-dates last major workflow change → move to `Archive/` subfolder within the workflow folder
  - If still active reference material → leave in place
  - `auto-prompts/` or similar output folders: keep the folder, archive contents where the corresponding remediation is closed
  - Note: folder hygiene is not a hard activation blocker but must be dispatched to CC before June 1

**L5 — Architecture + remediation**
- No open `haios_remediation_log` entries (status IN ('pending','in_progress')) for this workflow
- Not superseded by a CF Worker or newer workflow version
- Trigger type review: flag polling→event-driven migration candidates for post-June-1 (not a hard block unless Jordan decides)

**L6 — Post-reactivation verification**
- Activate in n8n
- Trigger one execution (manually trigger schedules with cadence > 5min)
- Execution completes with no errors
- haios_health_checks row appears, state='ok'
- **First post-activation execution is the credential canary of record** — a credential-auth failure here (as opposed to a logic error) means the L2 boundary canary was skipped or a credential went stale between canary and activation; deactivate, re-run the L2 authenticated read, then retry
- Slack alert fires if wired
- **E2E topic or record INSERT:** Before inserting test data into a Supabase table, query `information_schema.columns WHERE table_name = '{table}'` to verify column names, types, and constraint values (especially enum/CHECK fields). Schema drift between docs and live DB is common; schema mismatch causes immediate INSERT failure (Note: `thought-leadership` vs. `thought_leadership`, non-existent `slug` column). Takes 10 seconds; saves a failed-insert debug cycle.

**Rollback rule:** If L6 fails, deactivate immediately. Log to `haios_remediation_log`. Never leave a failing workflow active.

### 2.5. NOT-IN-PROD HANDLING (when workflow is being parked, not reactivated)

> **⚠ RUN THIS SECTION WHENEVER A WORKFLOW IS PARKED, DEACTIVATED, DECOMMISSIONED, OR MARKED Not-In-Prod — not only during a reactivation pass.** The registry suppression row below is mandatory and time-boxed (1–2 days). Omitting it produces a guaranteed `silent_failure` false positive.

When a workflow is confirmed Not-In-Prod for the current cycle:

**Skip:** L6 (activation). Gates L1–L5 are informational only.

**Required:** Monitoring suppression via `haios_workflow_registry`.

Full doctrine: `AOS/Domains/Infra/HEALTH-CHECK-PATTERNS.md` § "Monitoring Configuration — haios_workflow_registry"

**Summary:** The Remediation Trigger (`{{WORKFLOW_ID}}`) silently-failure-monitors any workflow that has written `ok` health checks in the last 7 days. A parked workflow goes dark — generating a false positive `silent_failure` remediation unless suppressed. Suppression = a row in `haios_workflow_registry` with `liveness_window_hours = NULL`.

**Steps:**
1. Check: `SELECT * FROM haios_workflow_registry WHERE workflow_id = '{id}'` via safe-sql
2. No row → INSERT `(workflow_id, liveness_window_hours) VALUES ('{id}', NULL)` — adapt to actual schema
3. Row exists → UPDATE `SET liveness_window_hours = NULL WHERE workflow_id = '{id}'`
4. Confirm via SELECT after write
5. Log in REACTIVATION-LOG: status = `⏸ NOT-IN-PROD`, note suppression complete

**Timing:** Complete within 1–2 days of the parking decision to avoid the 7-day false-positive window.

**When reactivating a parked workflow later:** DELETE the registry row or UPDATE to the appropriate `liveness_window_hours` value to restore monitoring.

**Not-In-Prod does NOT require:** snapshot, description fix, or L6 verification.

### 3. CC DISPATCH (when L3/L4 fixes needed)

Read `cc-prompt-skill`. Batch all mechanical fixes for a workflow into one CC task:
- typeVersion bumps
- haios_health_checks wiring additions
- README create/update
- Snapshot export + save to Drive

**Single-dispatch rule:** Build the complete cascade for the ENTIRE wave before dispatching — including the description batch for all NO-DESC workflows in that wave. Do not create a separate desc-batch prompt after structural fixes. Wave 1 required 2 CC dispatches because the desc batch was omitted from the initial cascade. Cost: one extra dispatch + false-flag investigation. Prevention: read the REACTIVATION-LOG §Critical Pre-Read + all per-workflow rows for the wave before writing the cascade, and include desc batch as a standing task.

Do not activate until CC confirms fixes applied and COO verifies.

### 4. UPDATE TRACKER + DOCS

After L6 passes for each workflow:
- Mark ACTIVE in REACTIVATION-LOG
- Update EXECUTION-BUDGET-LEDGER.md running total
- If credential spot-checked: update CREDENTIAL-MAP.md §1 last-verified date

### 5. SESSION CLOSE

- Confirm running cap total matches ledger sum
- Update session-context.md: N ACTIVE, M HOLD (blockers noted)
- Any HOLD workflows → NEXT SESSION queue with explicit blocker

## Handoff Contract

When complete:
- `AOS\Domains\Infra\REACTIVATION-LOG-{date}.md` — full per-workflow status
- Cap math: X/15,000 exec/mo scheduled (Y%)
- ACTIVE: N | HOLD: M (reasons) | SKIP-ARCHIVED: 2
- CC tasks dispatched (if any): list paths

## Dependencies

- Required: `EXECUTION-BUDGET-LEDGER.md` v1.0+ (confirmed cadences)
- Required: `CREDENTIAL-MAP.md` §1
- Required: n8n MCP (`n8n:get_workflow_details`)
- Required: Google Drive search (README/snapshot verification)
- Chains from: Finalize EXECUTION-BUDGET-LEDGER.md
- Chains to: `cc-prompt-skill` (mechanical fixes), `post-mortem-skill` (if L6 incident)

## Authority

Jordan dispatching the June 1 CC prompt IS the Tier 1 approval for all L6 activations within it. CC executes L1–L6 autonomously within the prompt scope. Daily Digest monitors health checks post-activation — no per-workflow Jordan sign-off needed.

COO pre-clearance (L1–5) = Tier 3 (Autonomous).

## Error Handling

- **L1 DEGRADED:** Log, skip, add to POST-JUNE-1 HOLD queue
- **L2 credential expired:** Log, skip, add to credential remediation queue
- **L3/L4 structural failures:** Dispatch CC, re-inspect next session
- **L5 open remediation entry:** Escalate to Jordan — document decision before proceeding
- **L6 execution error:** Deactivate immediately, log to `haios_remediation_log`, investigate before retry
- **n8n MCP unavailable:** Use REST: `GET https://{{N8N_HOST}}/api/v1/workflows/{id}`

## Cross-References

- `AOS/Domains/Infra/EXECUTION-BUDGET-LEDGER.md` — L1
- `AOS/Domains/Infra/CREDENTIAL-MAP.md` — L2
- `references/GATE-CRITERIA.md` — detailed pass/fail tests
- `references/WORKFLOW-LAYER-MAP.md` — workflow → Drive folder + wave order
- `cc-prompt-skill` — mechanical fix dispatch
- `post-mortem-skill` — L6 incident response
