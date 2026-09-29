---
name: n8n-workflow-audit
description: "Audit an n8n workflow against the 65-point quality checklist. Activates when reviewing, scoring, or verifying a workflow's production readiness."
---

# n8n Workflow Audit

Score a workflow against the 65-point quality checklist. Produces structured JSON the orchestrator can parse for pass/fail decisions.

**Scoring criteria:** @references/n8n-audit-criteria.md

## Audit Protocol

### 1. READ WORKFLOW
Fetch the workflow JSON via REST API:
```bash
KEY=$(cat ".secrets/n8n-api-key.txt" | tr -d '\n\r')
curl -s -H "X-N8N-API-KEY: $KEY" \
  "https://{{N8N_HOST}}/api/v1/workflows/{id}"
```

### 2. PHASE 1 — JSON ANALYSIS (43 pts)
Score from the workflow JSON alone. No external calls needed.
Evaluate: Triggers, Strategy, Metrics, Tech Stack, Cadence, Parameters, Error Handling.
Record evidence for each criterion.

### 3. PHASE 2 — RUNTIME VERIFICATION (22 pts)
Requires external checks:
- **Executions:** `GET /api/v1/executions?workflowId={id}&limit=10`
- **Database:** `POST /webhook/{{WEBHOOK_ID}}` to verify tables and records
- **Docs:** Read README from workflow directory on Drive
- **Semantic search:** `POST /webhook/{{WEBHOOK_ID}}` to check indexing

Score: Outputs, Database, Testing, Docs, Indexed.

### 4. COO REMINDERS (unscored)
Include in recommendations array — not scored, but COO needs to know:
- MCP access re-toggle needed after API edits
- Webhook callable by COO
- Outputs queryable via safe-sql

### 5. OUTPUT
Return structured JSON to the orchestrator:

```json
{
  "workflow_id": "{id}",
  "workflow_name": "{name}",
  "score": 48,
  "max_score": 65,
  "pass": true,
  "threshold": 48,
  "phase1_score": 30,
  "phase1_max": 43,
  "phase2_score": 18,
  "phase2_max": 22,
  "findings": [
    {"category": "M", "criterion": "Error alerts", "score": 0, "max": 2, "note": "No error alerting mechanism found"}
  ],
  "blockers": ["No Error Trigger node — required for production"],
  "recommendations": ["Add retryOnFail on HTTP nodes", "COO: re-toggle MCP access in n8n UI"],
  "coo_action_items": ["Verify webhook callable", "Test safe-sql query access", "Re-toggle MCP in n8n UI"]
}
```

## Scoring Thresholds

| Score | Status | Action |
|---|---|---|
| ≥48/65 (~74%) | PASS | Production ready — orchestrator triggers post-build flow |
| 36-47 (60-79%) | CONDITIONAL | Fix critical issues, re-audit |
| <36 (<60%) | FAIL | Major work needed, escalate |

## Rules

- Audit with fresh context. Never audit a workflow you just built — builder bias contaminates judgment.
- Score every criterion, even if 0. No skipping.
- Evidence is required for each score. "Looks fine" is not evidence.
- `findings` array includes ALL scored criteria below max, not just failures.
- `blockers` are issues that must be fixed before activation, regardless of total score.
- `coo_action_items` are things COO/Jordan need to handle (not CC's job).
