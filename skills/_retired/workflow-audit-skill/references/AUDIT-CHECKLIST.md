# Workflow Audit Checklist

Two phases: JSON Analysis (35 pts) + Runtime Verification (27 pts) = 62 total.

## Scoring Thresholds

| Score | Status | Action |
|---|---|---|
| ≥50/62 (80%) | ✅ PASS | Production ready |
| 39-49 (63-79%) | ⚠️ CONDITIONAL | Fix critical issues first |
| <39 (<63%) | ❌ FAIL | Major work needed |

---

# PHASE 1: JSON ANALYSIS (35 pts)

*Tool: `n8n:get_workflow_details` — parse returned JSON*

---

## A: Triggers (6 pts)

| Pts | Criteria |
|---|---|
| 2 | Exactly one trigger node |
| 2 | Type appropriate (webhook/schedule/manual) |
| 1 | Schedule expression valid (if scheduled) |
| 1 | Webhook path follows convention (if webhook) |

**Evidence:** Trigger node name, type, schedule/path

---

## D: Strategy (3 pts)

| Pts | Criteria |
|---|---|
| 1 | Purpose matches domain definition |
| 1 | Capability listed in registry |
| 1 | Layer assignment correct |

**Evidence:** Domain, capability, layer justification

---

## G: Metrics (4 pts)

| Pts | Criteria |
|---|---|
| 2 | Logs to health table (Supabase/Postgres node present) |
| 1 | Includes workflow_id in log |
| 1 | Logs success and failure paths |

**Evidence:** Logging node name, table name

---

## I: Tech Stack (3 pts)

| Pts | Criteria |
|---|---|
| 1 | All nodes standard or known community |
| 1 | No deprecated versions |
| 1 | Credentials use n8n store (not hardcoded) |

**Evidence:** Node types list, credential refs

---

## J: Cadence (5 pts)

| Pts | Criteria |
|---|---|
| 2 | Frequency matches purpose |
| 2 | No conflicts with related workflows |
| 1 | Schedule documented in workflow notes |

**Evidence:** Cron expression, purpose match

---

## L: Parameters (5 pts)

| Pts | Criteria |
|---|---|
| 2 | No hardcoded secrets |
| 2 | Config values use variables/credentials |
| 1 | Magic numbers documented |

**Red flags:** `Bearer sk-...`, hardcoded URLs, unexplained thresholds

**Evidence:** Parameter scan results

---

## M: Error Handling (9 pts)

| Pts | Criteria |
|---|---|
| 2 | `onError` on critical nodes |
| 2 | `retryOnFail` where appropriate |
| 2 | Error alerts (Slack/email/log) |
| 1 | Graceful degradation path |
| 2 | **Error Trigger → haios_health_checks** (universal pattern) |

**Evidence:** Error config on nodes, alert mechanism, Error Trigger node presence

---

# PHASE 2: RUNTIME VERIFICATION (27 pts)

*Requires Phase 1 complete. Uses external tools.*

---

## B: Outputs (5 pts)

**Tool:** `n8n:list_executions(workflowId, limit=10)`

| Pts | Criteria |
|---|---|
| 2 | Executions in last 7 days |
| 2 | Success rate ≥80% |
| 1 | No stuck executions >1 hour |

**Score 0 if no executions** — workflow untested

**Evidence:** Execution count, success rate, last run

---

## C: Database (4 pts)

**Tool:** Safe SQL webhook (`/webhook/{{WEBHOOK_ID}}`)

| Pts | Criteria |
|---|---|
| 2 | Tables workflow writes to exist |
| 1 | Tables have recent records |
| 1 | No test data pollution |

**Evidence:** Tables used, record counts

---

## E: Testing (4 pts)

**Tool:** `n8n:execute_workflow` (if safe) or `n8n:n8n_trigger_webhook_workflow`

| Pts | Criteria |
|---|---|
| 2 | Executes without errors |
| 1 | Output matches expected format |
| 1 | No production side effects |

**Skip if:** Destructive actions, no webhook, Jordan requests skip

**Evidence:** Execution ID + output, or skip reason

---

## F: Docs (4 pts)

**Tool:** Desktop Commander (`Filesystem:read_file`)

| Pts | Criteria |
|---|---|
| 2 | WORKFLOW-SPEC.md exists in correct folder |
| 1 | Includes purpose, trigger, inputs/outputs |
| 1 | Updated within 30 days or matches workflow |

**Evidence:** File path, last modified, content check

---

## H: COO Access (5 pts)

**Tool:** Verify via test call + Safe SQL

| Pts | Criteria |
|---|---|
| 2 | Webhook callable (if webhook) |
| 2 | Outputs queryable via Safe SQL |
| 1 | No auth blocking COO |

**Evidence:** Webhook URL, output table, sample query

---

## K: Indexed (5 pts)

**Tool:** Semantic search webhook (`/webhook/{{WEBHOOK_ID}}`)

| Pts | Criteria |
|---|---|
| 2 | WORKFLOW-SPEC.md indexed |
| 2 | Searchable by name and purpose |
| 1 | Returns in top 5 results |

**Evidence:** Search rank, chunk content
