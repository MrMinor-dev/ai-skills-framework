# n8n Workflow Audit Criteria
<!--: +3 enforcement checks — Code node mode blocker, HC field-value blocker, name-format §N expansion -->

65-point checklist in two phases. Score every criterion — no skipping.

---

## BLOCKERS (Silent-Correctness Defects)

Run these checks BEFORE scoring. A BLOCKER fails the audit regardless of point total — do not activate a workflow with an open BLOCKER.

| Blocker | Check | Finding format |
|---|---|---|
| **Code node mode** | Every `n8n-nodes-base.code` node must have `parameters.mode == "runOnceForAllItems"` | `BLOCKER: Code node {name} missing runOnceForAllItems — silent-wrong-output crash` |
| **HC field values** | Every `haios_health_checks` INSERT: `layer` must be one of `HAIOS`, `AOS`, `Agency` (NOT `L2` or similar maturity tag); `business_id` must be `mrminor` | `BLOCKER: HC layer/business_id wrong —` |
| **Name format** | Workflow name must match `^(HAIOS\|AOS\|Agency): .+ - .+ \[L[0-4]\]$` with no `Vx` suffix | `BLOCKER: Name format unparseable — cannot determine Layer/maturity` |

---

## PHASE 1: JSON ANALYSIS (43 pts)

Score from the workflow JSON returned by `GET /api/v1/workflows/{id}`.

### A: Triggers (6 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Exactly one trigger node | Trigger node name + type |
| 2 | Type appropriate for use case (webhook/schedule/manual) | Type + justification |
| 1 | Schedule expression valid (if scheduled) | Cron expression |
| 1 | Webhook path follows convention (if webhook) | Path value |

### D: Strategy (3 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 1 | Purpose matches AOS domain definition | Domain name |
| 1 | Capability listed in SERVICE-CATALOG.md | Catalog entry or "missing" |
| 1 | Layer assignment correct (AOS vs niche) | Layer + reasoning |

### G: Metrics (4 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Logs to `haios_health_checks` table | Logging node name + table |
| 1 | Includes `workflow_id` in health check log | Field present in node config |
| 1 | Logs both success and error paths | Both paths traced |

### I: Tech Stack (3 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 1 | All nodes standard or known community types | Node types list |
| 1 | No deprecated node versions | typeVersion values checked |
| 1 | Credentials use n8n credential store (not hardcoded) | Credential refs, no inline secrets |

### J: Cadence (5 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Frequency matches purpose | Cron + purpose match |
| 2 | No schedule conflicts with related workflows | Related workflow schedules checked |
| 1 | Schedule documented in workflow notes or README | Documentation location |

### L: Parameters (5 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | No hardcoded secrets in node parameters | Scan results (grep for `sk-`, `Bearer`, API keys) |
| 2 | Config values use credentials or variables | Config approach |
| 1 | Magic numbers documented or explained | Any unexplained thresholds flagged |

### M: Error Handling (12 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 3 | Error Trigger node present | Node name or "MISSING — BLOCKER" |
| 2 | `onError: continueErrorOutput` on critical nodes | Nodes with error config listed |
| 2 | Error output paths connected (not dangling) | Connection verification |
| 2 | `retryOnFail` on HTTP/external API nodes | Retry config per node |
| 2 | Error alerts log to `haios_health_checks` | Error path traces to logging |
| 1 | Graceful degradation path exists | Degradation strategy |

### N: Remediation Maturity (5 pts)

| Pts | Criterion | Evidence |
|---|---|---|
| 1 | Health check logging on success path (L1+) | Postgres node writing `state: 'ok'` to `haios_health_checks` |
| 1 | Error Trigger present and logging errors (L2+) | Error Trigger node -> Postgres node writing `state: 'error'` |
| 1 | Error Trigger connected to Remediation Trigger webhook (L3) | HTTP Request node in Error path calling `/webhook/{{WEBHOOK_ID}}` |
| 1 | Workflow name matches full convention | Name matches pattern `(HAIOS/AOS/Agency): Domain - Capability [L#]` with no Vx suffix; finding if non-compliant, BLOCKER if Layer/format unparseable (see BLOCKERS above) |
| 1 | JSON backup exists in workflow Drive folder | `workflow.json` file present and recent |

---

## PHASE 2: RUNTIME VERIFICATION (22 pts)

Requires external checks against live systems.

### B: Outputs (5 pts)

Check via: `GET /api/v1/executions?workflowId={id}&limit=10`

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Executions exist in last 7 days | Execution count + dates |
| 2 | Success rate ≥80% | Pass/fail ratio |
| 1 | No stuck executions >1 hour | Execution durations |

Score 0 for all if no executions exist (workflow untested).

### C: Database (4 pts)

Check via: `POST /webhook/{{WEBHOOK_ID}}`

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Tables workflow writes to exist | Table names verified |
| 1 | Tables have recent records | Record counts + dates |
| 1 | No test data pollution | Test records absent or cleaned |

### E: Testing (4 pts)

Check via: webhook trigger or execution review.

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | Executes without errors | Execution ID + status |
| 1 | Output matches expected format | Output shape verification |
| 1 | No unintended production side effects | Side effect check |

Skip with documented reason if: destructive actions, no webhook, safety concern.

### F: Docs (4 pts)

Check via: read README from workflow directory on Drive.

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | README exists in correct folder | File path |
| 1 | Includes purpose, trigger, inputs/outputs | Content check |
| 1 | Updated within 30 days or matches current workflow | Last modified date |

### K: Indexed (5 pts)

Check via: `POST /webhook/{{WEBHOOK_ID}}`

| Pts | Criterion | Evidence |
|---|---|---|
| 2 | README indexed in semantic search | Search result |
| 2 | Searchable by workflow name and purpose | Search queries + results |
| 1 | Returns in top 5 results for relevant query | Search rank |

---

## UNSCORED: COO REMINDERS

These are not scored criteria. Include in the `recommendations` array for COO awareness.

### COO Access Check
Remind COO to verify after build:
- Webhook callable by COO (if webhook-triggered)
- Outputs queryable via safe-sql
- MCP access re-toggled in n8n UI if API edits were made
- No auth blocking COO access to workflow or outputs
