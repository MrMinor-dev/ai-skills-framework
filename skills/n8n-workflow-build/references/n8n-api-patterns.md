# n8n REST API Patterns for CC

Exact curl patterns for workflow operations. CC uses REST API exclusively (no MCP tools).

> **Conventions, rules, and what-to-do-when live in CORE: `~/.claude/rules/n8n-workflows-core.md`.** This file is syntax-only. Cross-refs point to CORE where the "why" belongs there.

## Setup

```bash
KEY=$(cat ".secrets/n8n-api-key.txt" | tr -d '\n\r')
BASE="https://{{N8N_HOST}}/api/v1"
```

## Read Workflow

```bash
curl -s -H "X-N8N-API-KEY: $KEY" "$BASE/workflows/{id}"
```

## Create Workflow

```bash
curl -s -X POST -H "X-N8N-API-KEY: $KEY" -H "Content-Type: application/json" \
  "$BASE/workflows" \
  -d '{"name": "...", "nodes": [...], "connections": {...}, "settings": {}}'
```

## Update Workflow (Full PUT — the ONLY mutation method)

See CORE § Mutation Rules for whitelist fields, MCP preservation, connection format, etc. Canonical syntax:

```bash
# 1. GET current state
curl -s -H "X-N8N-API-KEY: $KEY" "$BASE/workflows/{id}" > /tmp/workflow.json

# 2. Modify in Python + filter to whitelist. Write PUT body to a file.
#    (Large bodies MUST use @filepath; inline `-d` breaks at ~32KB on Windows. CORE § Workflow Task Operations.)
python3 -c "
import sys, json
w = json.load(open('/tmp/workflow.json'))
# ... modify w['nodes'], w['connections'] ...

# Body whitelist: only name/nodes/connections/settings. NO staticData, NO triggerCount.
# Settings whitelist: only executionOrder, callerPolicy, availableInMCP. (Raw GET settings cause 400.)
put_body = {
    'name': w['name'],
    'nodes': w['nodes'],
    'connections': w['connections'],
    'settings': {k: w['settings'][k] for k in ('executionOrder', 'callerPolicy', 'availableInMCP')
                 if k in w.get('settings', {})},
}
put_body['settings'].setdefault('availableInMCP', True)   # ALWAYS preserve MCP access
json.dump(put_body, open('/tmp/put_body.json', 'w'))
"

# 3. PUT with @filepath
curl -s -X PUT -H "X-N8N-API-KEY: $KEY" -H "Content-Type: application/json" \
  "$BASE/workflows/{id}" -d @/tmp/put_body.json
```

**Windows temp path:** `~/AppData/Local/Temp/put_body.json`.

## Activate / Deactivate

```bash
curl -s -X POST -H "X-N8N-API-KEY: $KEY" "$BASE/workflows/{id}/activate"
curl -s -X POST -H "X-N8N-API-KEY: $KEY" "$BASE/workflows/{id}/deactivate"
```

## Trigger Webhook

```bash
curl -s -X POST "https://{{N8N_HOST}}/webhook/{path}" \
  -H "Content-Type: application/json" \
  -d '{"test": true, ...}'
```

Rules (activate-first, no `/webhook-test/`, multi-trigger behavior): CORE § Webhook Testing + § Multi-Trigger Workflows.

## Check Execution Results

```bash
curl -s -H "X-N8N-API-KEY: $KEY" \
  "$BASE/executions?workflowId={id}&limit=5&includeData=true"
```

Execution data lives at `data.resultData.runData["{nodeName}"]`. Each node has `data.main[0]` (success items) and optionally `data.main[1]` (error items when `onError: continueErrorOutput`). Diagnostic methodology: CORE § Diagnosis.

## Verify DB Output (safe-sql webhook)

```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT * FROM {table} ORDER BY created_at DESC LIMIT 5"}'
```

Response format: markdown table inside a `response` field, NOT JSON rows. CORE § Workflow Task Operations has the parsing gotcha.

## Supabase REST Cleanup (service role key required)

```bash
SUPA_KEY=$(cat ".secrets/supabase-service-role.txt" | tr -d '\n\r')
curl -s -X DELETE \
  "{{SUPABASE_URL}}/rest/v1/{table}?{filters}" \
  -H "apikey: $SUPA_KEY" -H "Authorization: Bearer $SUPA_KEY"
```

## Node Structure Template

```json
{
  "id": "short-descriptive-id",
  "name": "Descriptive Name",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [500, 300],
  "parameters": {},
  "credentials": {}
}
```

- **Position grid:** x increments of 250, y of 200.
- **IDs:** short descriptive strings for new nodes; preserve existing IDs on updates.
- **Names:** must be unique within workflow.
- **typeVersion:** use exactly what the prompt spec provides. Wrong version = silent failures. Postgres cap is 2.5 on this instance (CORE § Mutation Rules).

## Connection Format

Standard (single output):

```json
{
  "Source Node Name": {
    "main": [[
      {"node": "Target Node Name", "type": "main", "index": 0}
    ]]
  }
}
```

Multi-output (IF, Switch — each inner array is one output port):

```json
{
  "IF Node": {
    "main": [
      [{"node": "True Branch", "type": "main", "index": 0}],
      [{"node": "False Branch", "type": "main", "index": 0}]
    ]
  }
}
```

Simplified `{"Source": [["Target"]]}` is NOT the API format (CORE § Mutation Rules). `splitInBatches` typeVersion 3 has inverted port order: 0 = done, 1 = loop body (CORE § Expression Safety).

## Database Connection (Supabase Session Pooler)

Direct Connection is IPv6-only and fails in n8n. Always use Session Pooler:

```
Host: aws-0-us-west-2.pooler.supabase.com
Database: postgres
User: postgres.[project-ref]
Port: 5432
SSL: DISABLED
```

Get exact values from: Supabase Dashboard → Connect → Session Pooler.

Symptoms of the wrong connection: IPv6 `ENETUNREACH`, self-signed certificate errors, "Tenant or user not found".

## Schema Verification Before INSERT/UPDATE

CHECK constraints: use CORE's canonical JOIN pattern (§ Mutation Rules). Supplementary queries for column/sample inspection:

```sql
-- Columns and types
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = '{target_table}';

-- Sample existing data
SELECT * FROM {target_table} LIMIT 3;
```
