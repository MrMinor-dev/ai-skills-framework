---
name: n8n-diagnose
description: "Diagnose and fix failing n8n workflows using a 4-phase method (Evidence → Diagnose → Fix → Verify). Activates when asked to debug, fix errors, or remediate any n8n workflow."
triggers:
  - diagnose workflow
  - fix workflow errors
  - workflow remediation
  - debug n8n workflow
---

# n8n Workflow Diagnose

Systematic remediation for any n8n workflow. Do NOT skip phases — premature fixes without evidence waste full cycles.

**API reference:** @AOS\Domains\Infra\N8N-API-REFERENCE.md
**Rollback rule:** @~/.claude/rules/n8n-rollback.md

---

## Phase 1: EVIDENCE

Gather all available data before hypothesizing.

```bash
KEY=$(cat ".secrets/n8n-api-key.txt" | tr -d '\n\r')
BASE="https://{{N8N_HOST}}/api/v1"
WF="{WORKFLOW_ID}"
```

**1. Health checks — recent errors:**
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"SELECT state, error_message, node_name, execution_id, timestamp FROM haios_health_checks WHERE workflow_id = '$WF' AND timestamp > NOW() - INTERVAL '7 days' ORDER BY timestamp DESC LIMIT 20\"}"
```

**2. Execution history — last 10 (with data):**
```bash
curl -s -H "X-N8N-API-KEY: $KEY" \
  "$BASE/executions?workflowId=$WF&limit=10&includeData=true"
```
Check: `status`, `mode` (manual/webhook/{{WEBHOOK_ID}}/**error**), `startedAt`, `stoppedAt`, which node failed, error message.

**3. Workflow definition:**
```bash
curl -s -H "X-N8N-API-KEY: $KEY" "$BASE/workflows/$WF"
```

**4. README** (if exists):
```
AOS/Workflows/{folder}/README.md
```

---

## Phase 2: DIAGNOSE

Identify the root cause from evidence. State your hypothesis explicitly before moving to Phase 3.

**Key patterns:**

| Pattern | Signal | What to look for |
|---|---|---|
| **Error Trigger cascade** | 2× error rate, executions with `mode: error` | Broken Error Trigger creates a second failed execution for every main failure — doubles apparent error count |
| **Silent failure** | status=success, 0 rows written | Filter/conditional matching nothing — trace data flow node by node |
| **Naming mismatch** | Regex/expression fails | Pattern matched old names (e.g. `Agent N`) but not current names (`AOS:`, `HAIOS:`, `a former e-commerce project:`) |
| **Credential issue** | 401/403 on specific node | Check after credential rotation |
| **Schema drift** | Column not found, type mismatch | Workflow writes to a column that was renamed or removed — verify with `safe-sql` schema query |

If health check nodes are failing, query the schema directly:
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT column_name, data_type, is_nullable FROM information_schema.columns WHERE table_name = '\''haios_health_checks'\'' ORDER BY ordinal_position"}'
```

---

## Phase 3: FIX

**Before any PUT:** save a rollback snapshot (`GET` the workflow, write to a temp file).

**Whitelist PUT only** — never build from scratch, never use blacklist stripping:
1. `GET /api/v1/workflows/{id}` → full JSON
2. Modify only the broken node(s) in memory
3. Build PUT body with ONLY: `name`, `nodes`, `connections`, `settings`, `staticData`
4. `PUT /api/v1/workflows/{id}` with that body

**After every PUT, verify `availableInMCP`:**
```bash
echo $RESPONSE | python3 -c "import sys,json; w=json.load(sys.stdin); print('MCP:', w.get('settings',{}).get('availableInMCP','MISSING'))"
```
Missing `availableInMCP` silently disables MCP execution — no error, no warning.

**`staticData: null`** — include in PUT body when fixing trigger-related issues to clear stale polling state.

**Two-tier remediation:**
- **Tier A — auto-fix:** Node-level bugs: wrong regex, bad expression, incorrect field mapping, query parameter binding, credential reference. Fix directly.
- **Tier B — escalate:** Architecture changes: adding/removing nodes, changing connections, restructuring flow. Document the recommendation in the feedback file. Do NOT implement. Notify: *"This requires architecture changes — escalating to COO."*

---

## Phase 4: VERIFY

**1. Trigger execution** — use MCP `execute_workflow` (REST `/run` returns 405 on n8n Cloud):
```
mcp: execute_workflow(workflowId="{WORKFLOW_ID}")
```

**2. Check target tables** — query the tables the workflow writes to. Confirm new rows with correct data.

**3. Check health check:**
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"SELECT state, timestamp FROM haios_health_checks WHERE workflow_id = '$WF' ORDER BY timestamp DESC LIMIT 3\"}"
```
Expect: `state = 'ok'` with a recent timestamp.

**4. Run 2×** — trigger a second execution to confirm it's not a one-time fluke.

---

## Rules

- **Iteration cap:** 3 fix attempts maximum. After 3 failures, escalate to Jordan with the full evidence log.
- **One fix per cycle.** Multiple changes in one PUT mask which change helped or broke.
- **Never guess.** Read execution data. Hypothesize. Then fix.
- **MCP access warning:** After any REST API PUT, MCP access toggles OFF in n8n UI. Warn Jordan to re-enable.
