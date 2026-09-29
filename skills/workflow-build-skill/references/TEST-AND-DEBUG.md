# Phases 3-4: TEST & DEBUG

**Phase 3 Gate:** ONE input → correct output verified in DB/destination.
**Phase 4 Gate:** All bugs resolved, re-verified with clean run.

These phases are a tight loop: test → break → debug → fix → re-test. Loaded together because you'll bounce between them.

---

## TEST

### The #1 Rule
**Test with ONE input first.** Not 8 subreddits. Not 50 emails. ONE.

### Manual Execution
Webhook trigger:
```
n8n-mcp:n8n_trigger_webhook_workflow → webhookUrl, data
```
Schedule-only trigger → temporarily add a Manual Trigger node for testing.

### Verify Output (Not Just "Ran")
After execution:
```
n8n-mcp:n8n_list_executions → workflowId, limit: 1
n8n-mcp:n8n_get_execution → id, mode: 'summary'
```
**"Ran successfully" ≠ correct output.** Invoice Handler "succeeded" for many sessions before data was right.

Walk execution node-by-node:
- Did each node receive expected data?
- Did expressions resolve to values (not `undefined`, `[Object object]`, `null`)?
- Did DB write land? **Always verify with SQL:**
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT * FROM table_name ORDER BY created_at DESC LIMIT 5"}'
```

### Expression Danger Zones
Expressions are the #1 bug source. Every row below is a bug we actually hit:

| Trap | What Goes Wrong | Lesson Source |
|---|---|---|
| `$json` after DB node | Postgres UPSERT/INSERT returns different shape than input. `$json` now points to DB response, not your data. | Email Router |
| Missing `{{ }}` | Bare `$json.field` in parameter field doesn't evaluate — treated as literal string | Email Router |
| `$json` after merge/switch | `$json` may point to wrong upstream branch. Use `$('Node Name').item.json` to be explicit | Invoice Handler |
| Hardcoded where expression needed | `error_id: 0` instead of `{{ $now.toMillis }}` — static value when dynamic required | Email Router |
| JSONB expects object, not string | Passing `'{"key":"val"}'` (string) instead of `{"key":"val"}` (object) crashes Postgres | Email Router |
| DB constraint mismatch | Writing a value not in the column's enum/check constraint | Email Router |
| n8n node strips data | Gmail node removes `payload.parts` silently — only execution data reveals this | Invoice Handler |
| Referencing node that was renamed | Connections update but expressions with `$('Old Name')` don't auto-update | General pattern |

### Iteration Loop
Fix → re-deploy → re-test:
```
n8n-mcp:n8n_get_workflow → id              (read current state)
# Modify the specific node/expression
n8n-mcp:n8n_update_full_workflow → id       (write full state)
# Re-trigger and re-verify
```
**One fix per cycle.** Multiple fixes in one update mask which change helped or broke something new.

---

## DEBUG

### Execution Forensics (Always Start Here)
Don't guess. Read the data:
```
n8n-mcp:n8n_get_execution → id, mode: 'preview'    (structure + item counts — fast)
n8n-mcp:n8n_get_execution → id, mode: 'summary'     (2 sample items per node)
n8n-mcp:n8n_get_execution → id, mode: 'filtered', nodeNames: ['Suspect Node']  (deep look)
```

**Forensics workflow:**
1. `preview` → find which node errored or produced unexpected item count
2. `summary` → see actual data shape at key nodes
3. `filtered` → zoom into the suspect node's full input/output

### Common Failure Patterns

| Symptom | Likely Cause | Fix |
|---|---|---|
| Node receives empty `$json` | Upstream returned different shape than expected | Check execution data at UPSTREAM node |
| "Cannot read property X of undefined" | Expression references field that doesn't exist at runtime | `filtered` mode on source node to see actual shape |
| DB write "succeeds" but no rows appear | UPSERT matched existing, did nothing | Check ON CONFLICT clause; verify with SELECT count |
| Workflow errors on re-run only | Duplicate key constraint from previous test data | Add ON CONFLICT DO NOTHING, or clean test data first |
| Wrong IF/Switch branch fires | Condition evaluates differently than expected | Add Set node BEFORE the IF to log actual value being tested |
| "Workflow could not be started" | typeVersion mismatch or invalid parameter structure | `n8n_validate_workflow` on the workflow ID |
| Correct data in, wrong data out | Code node logic error or expression transformation bug | `filtered` mode on both input AND output of the Code node |
| Webhook returns 404 | Workflow not active, or webhook path mismatch | Check workflow active status + webhook node path config |

### Isolate the Problem
1. `n8n_get_workflow → id` — read full workflow
2. Find suspect node's parameters
3. `get_node_essentials` — compare configured vs correct format
4. Fix ONE thing
5. Re-test ONE input
6. Repeat

### Autofix (Structural Issues Only)
For type versions, expression format, webhook paths:
```
n8n-mcp:n8n_autofix_workflow → id, applyFixes: false   (PREVIEW first)
```
Review. Then if appropriate:
```
n8n-mcp:n8n_autofix_workflow → id, applyFixes: true
```
Autofix handles structural issues. It does NOT fix logic bugs or wrong expressions.

---

## Token Awareness During Test/Debug

These phases burn the most tokens — execution data is verbose.

**Economize:**
- Use `mode: 'preview'` first (cheapest) to narrow the suspect
- Use `mode: 'filtered'` with specific `nodeNames` instead of `mode: 'full'`
- Don't load the full workflow JSON repeatedly — load once, modify in memory
- If you've done 3+ test/debug cycles and context is growing, consider checkpointing

**Checkpoint trigger:** If >60% context consumed during test/debug, stop and checkpoint per SKILL.md handoff procedure. Better to hand off cleanly than lose findings to compaction.

---

## Phase 3-4 Complete When:
- [ ] ONE input tested end-to-end
- [ ] Output verified in destination (not just "ran successfully")
- [ ] All expression references validated against actual execution data
- [ ] No constraint violations on DB writes
- [ ] Clean re-run produces expected results (idempotent)
