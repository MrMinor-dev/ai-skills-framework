# Phase 2: BUILD Patterns

**Gate:** Workflow exists in n8n, all nodes present, connections wired, validated.

## 2.1 Node Research (MANDATORY)
For EVERY node type you'll use:
```
n8n-mcp:get_node_essentials → nodeType    (ALWAYS FIRST — 5KB, shows required fields)
n8n-mcp:get_node_documentation → nodeType  (only if essentials insufficient)
```
**Never configure a node from memory.** Node APIs change between versions. 5 seconds to check saves hours of debugging.

Common node types (always verify version):
```
n8n-mcp:get_node_essentials({nodeType: "nodes-base.httpRequest"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.postgres"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.set"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.if"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.code"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.scheduleTrigger"})
n8n-mcp:get_node_essentials({nodeType: "nodes-base.manualTrigger"})
```

## 2.2 Build Order
1. **Trigger node first** — Schedule, Webhook, or Manual
2. **Happy path only** — no error handling yet, no edge cases
3. **Hardcoded test values** — use Set nodes with literal values, not expressions
4. **One node at a time** — add node, verify it appears, then next

## 2.3 Workflow Creation
```
n8n-mcp:n8n_create_workflow → name, nodes[], connections{}
```

### Node Structure (every node needs all of these):
```json
{
  "id": "uuid-format",
  "name": "Descriptive Name",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [500, 300],
  "parameters": {}
}
```

### Key Rules:
- **typeVersion:** Get from `get_node_essentials` — wrong version = silent failures
- **Position grid:** x increments of 250, y of 200 (keeps layout clean)
- **IDs:** Use UUID format. Generate with `crypto.randomUUID()` pattern or sequential.
- **Names:** Must be unique within workflow. Use descriptive names.

### Connection Format:
```json
{
  "Source Node Name": {
    "main": [[
      {"node": "Target Node Name", "type": "main", "index": 0}
    ]]
  }
}
```
For nodes with multiple outputs (IF, Switch):
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

## 2.4 Verify Scaffold
After creation:
```
n8n-mcp:n8n_get_workflow → id          (confirm nodes + connections match intent)
n8n-mcp:n8n_validate_workflow → id      (catch structural issues)
```
If validation errors → fix BEFORE proceeding. Don't carry structural debt into testing.

## 2.5 Editing the Workflow
**NEVER use `n8n_update_partial_workflow`** — unreliable, consistently fails.

Always:
```
n8n-mcp:n8n_get_workflow → id           (read current full state)
# Modify nodes/connections in memory
n8n-mcp:n8n_update_full_workflow → id, nodes, connections   (write full state back)
```

## 2.6 Proven n8n Node Patterns

Copy these patterns exactly — they come from verified working workflows.

### Error Trigger → Postgres (direct INSERT)
Every AOS workflow gets this. Error Trigger connects directly to a Postgres node. No Code, no HTTP.
```json
// Error Trigger node
{"type": "n8n-nodes-base.errorTrigger", "typeVersion": 1, "parameters": {}}

// Error: Log node (Postgres, direct SQL INSERT)
{"type": "n8n-nodes-base.postgres", "typeVersion": 2.5,
 "parameters": {
   "operation": "executeQuery",
   "query": "=INSERT INTO haios_health_checks (check_id, component, layer, state, error_message, timestamp, workflow_id) VALUES ('COMPONENT-err_' || to_char(NOW(), 'YYYYMMDD_HH24MISS'), 'COMPONENT-NAME', 'aos', 'error', '{{ $json.execution.error.message }}', NOW(), 'WORKFLOW-ID')"
 },
 "retryOnFail": true, "waitBetweenTries": 5000}
```
Replace `COMPONENT-NAME` and `WORKFLOW-ID` with actual values.

### If Node v2 — Condition on Data Presence
Don't use `boolean` condition type. Check the field that matters using `string` → `notEmpty`:
```json
{"conditions": {
  "options": {"caseSensitive": true, "typeValidation": "strict"},
  "conditions": [{"leftValue": "={{ $json.query }}", "rightValue": "", "operator": {"type": "string", "operation": "notEmpty"}}],
  "combinator": "and"}}
```

### Code Node v2 — Single Output Only
Code node v2 returns `[{json: {...}}]`. It does NOT support multi-output arrays like `[[], [{json}]]`. For branching, use an If or Switch node after the Code node.

### Postgres with Error Catch
When Postgres errors should be handled gracefully (not crash the workflow):
```json
{"type": "n8n-nodes-base.postgres", "typeVersion": 2.5,
 "onError": "continueErrorOutput",
 "retryOnFail": true, "maxTries": 2, "waitBetweenTries": 2000}
```
Connect output 0 → success path, output 1 → error handler (Code node to format error message).

### Credentials Warning
`n8n_update_full_workflow` drops all credential assignments. After every API edit:
1. Warn Jordan: "MCP access OFF + credentials need re-adding on [node names]"
2. List the specific nodes that need credentials

## Phase 2 Complete When:
- [ ] All node types researched via `get_node_essentials`
- [ ] Workflow created in n8n with all nodes + connections
- [ ] `n8n_validate_workflow` passes (no errors)
- [ ] Workflow ID recorded in README
- [ ] Happy path only — no error handling nodes yet
