---
name: database-skill
description: "Execute database operations. Triggers: create table, modify schema, SQL operations."
---

# Database Skill

Execute database operations. Live DB is authoritative — no index doc to maintain.

## Workflow

### 1. DETERMINE OPERATION TYPE

| Type | Examples | Method |
|---|---|---|
| **DDL** | CREATE TABLE, ALTER, DROP | Generate SQL → Jordan runs in Supabase |
| **DML** | INSERT, UPDATE, DELETE | Safe DB Write webhook |
| **Query** | SELECT | Safe SQL Query webhook |

**DDL blocked by Safe DB Write** (by design — schema changes = Tier 1).

### 2. EXECUTE

**For DDL (CREATE/ALTER/DROP):**
```
Generate SQL with namespace prefix:
- haios_ = HAIOS (meta layer)
- aos_ = AOS (platform layer)
- agency_ = Agency (MRMINOR operations)
- {client}_ = Client-specific table (if needed)

If layer placement is unclear, consult the decision tree at:
  doc-management-skill/references/LAYER-TAXONOMY.md

Include COMMENT ON TABLE with purpose description.
Present SQL to Jordan → they run in Supabase SQL Editor.
Wait for confirmation: "done", "created", "success"
```

**For DML (INSERT/UPDATE/DELETE):**
URL: `https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}}`
```json
{"query": "[SQL statement]"}
```
⚠️ **Table whitelist enforced — 16 tables, live-verified by a rejected write, not copied from a doc.** safe-db-write only accepts writes to: `policy_alerts`, `tool_updates`, `search_trends`, `system_flags`, `haios_health_checks`, `optimization_recommendations`, `haios_workflow_registry`, `haios_messages`, `haios_remediation_log`, `retail_raw_feedback`, `retail_interviews`, `retail_interview_signals`, `aos_intel_analysis`, `aos_expenses`, `aos_intel_sources`, `aos_wiki_lint_log`. **The rejection message enumerates the live list — it is the authoritative instrument; this line is a convenience copy and rots.** (correction: this list said `health_checks` — **no such table**, the live entry is `haios_health_checks` — and omitted `aos_wiki_lint_log`. 15 listed vs 16 live, one misnamed.) **`agency_pipeline` is NOT whitelisted** — route via Jordan (Supabase SQL Editor) or CC psycopg2 direct (in N8N-API-REFERENCE.md Pitfall #35). (confirmed — canonicalized — re-verified live.) **`aos_tax_config` is NOT whitelisted (tested)** — tax config writes go to Jordan.
⚠️ **Rows affected count unreliable for multi-row updates** — webhook reports `1 row` even when N rows changed. Always verify via a follow-up SELECT.

**For Query (SELECT):**
URL: `https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}}`
```json
{"query": "[SELECT statement]"}
```

**For column details (always available):**
```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name = '{table}' ORDER BY ordinal_position
```

**For table purpose:**
```sql
SELECT obj_description('{table}'::regclass)
```

### 3. REPORT CHANGED FILES

List any docs modified this operation for orchestrator.
Orchestrator decides if reindex needed.

## Handoff Contract

When complete, state:
- Operation: [DDL/DML type]
- Tables affected: [list]
- Reindex: [triggered / batched for session end]

## Dependencies

- Required: Safe DB Write webhook, Safe SQL Query webhook
- Schema: query `information_schema` or `pg_tables` directly — live DB is authoritative. No index doc.

## Authority

- DDL (CREATE/ALTER/DROP): **Tier 1** — Jordan runs directly
- DML (INSERT/UPDATE/DELETE): **Tier 2** — via Safe DB Write webhook
- SELECT: **Tier 3** — autonomous

## Error Handling

| Issue | Action |
|---|---|
| Safe DB Write rejects | Check whitelist in workflow — may need DDL instead |
| Table already exists | Verify with SELECT, ask Jordan to confirm intent |
| `Column 'X' not allowed for UPDATE` | safe-db-write enforces a **per-table column whitelist** (narrower than the table whitelist). The error lists the allowed columns. For `aos_intel_analysis`: only `action_status, action_required, action_items, class, theme, confidence` — the `disposition_*` columns are **pipeline-only by design**; encode review state in `action_status` (`dismissed` / `reviewed`) + `action_required`. |
| `Column 'X' not allowed` where X is a **string value**, not a column | The validator token-matches quoted args and false-positives on values inside `jsonb_build_object(...)` (e.g. flagged `'eval'`). Build jsonb from a plain literal instead: `action_items='["note text"]'::jsonb`, not `jsonb_build_object('k','eval')`. |
