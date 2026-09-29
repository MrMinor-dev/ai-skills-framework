# MIGRATE Task Type

Database, data, schema.

**Recommended effort:** `high`

- Override to `xhigh` for schema-complex migrations with many edge cases or behavioral risk
- Override to `medium` for straightforward value transforms (static lookup, one-to-one rename)

<!-- COO uses this file when writing a CC prompt for a MIGRATE task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## MIGRATION
- **Source:** {table/schema/file}
- **Target:** {table/schema/file}
- **Mapping:** {how data transforms}
- **Constraints:** {what must be preserved}

## ROLLBACK
{How to undo the migration.}

## VERIFICATION
{Queries to confirm migration succeeded.}
```

## Session Learnings

**Cross-ref grep scope for file-move tasks:** Full-tree recursive grep over all MRMINOR/.md files is a disproportionate time sink for file-move tasks where moved docs have low external linkage. MIGRATE-ARCHITECTURE-CLEANUP: moves + verification took 2 min; 6× full-tree grep ran 18+ min before interruption. DAILY-DIGEST-TUNING and REMEDIATION-NOISE had zero external cross-refs — those two passes were pure waste. **Fix:** At prompt-write time, COO identifies candidate cross-ref holders (session-context.md, BUILD-ROADMAP.md, SSOT-REGISTRY-MASTER.md, COO-REFERENCE-MANUAL.md, source/target workflow READMEs, SSOT index docs) and targets those first. Full-tree grep is fallback only if candidate scan comes up clean. For low-linkage docs (session-suffix names like `.md`, tuning logs, regression audits, archived outputs): skip full-tree grep entirely; note omission in build log. See the anti-pattern log.

**`pg_policies` view inaccessible via safe-sql webhook:** MIGRATE prompts that instruct CC to query `pg_policies` to copy an existing RLS policy pattern will fail — the view returns "column polname does not exist" via the safe-sql webhook (access restriction in the pooled connection context). Use `pg_catalog.pg_policy JOIN pg_catalog.pg_class` instead: `SELECT p.polname, p.polcmd FROM pg_catalog.pg_policy p JOIN pg_catalog.pg_class c ON c.oid = p.polrelid WHERE c.relname = '{table}'`. For MIGRATE prompts: either embed the `pg_catalog` query form directly, or state the standard service-role bypass pattern inline (`TO service_role USING (true) WITH CHECK (true)`) so CC doesn't need to discover it at runtime.

**Data cleanup / backfill count verification:** When verifying a data cleanup task (e.g., renaming column values via UPDATE/PATCH), target the "zero non-canonical rows" assertion -- NOT an exact post-UPDATE count match. Post-UPDATE counts for renamed values will be HIGHER than the spec's pre-cleanup count if the workflow was already writing canonical values concurrently (old rows get merged into an existing canonical bucket). Write the VERIFICATION section as: "Should return 0 rows" for the non-canonical check, not "Should show total = N." Mismatching a bucket count is correct behavior, not a bug.

**safe-sql is SELECT-only — DDL execution path must be specified:** The safe-sql webhook blocks all non-SELECT statements (`ALTER`, `CREATE`, `DROP`, etc.) with `❌ Blocked: ALTER not allowed. SELECT only.` For any MIGRATE prompt that includes DDL, the §SPEC or §9 EXECUTION POLICY MUST specify the execution path. Four options in order of CC autonomy:
1. **n8n one-off workflow** (full CC autonomy) — CC builds a temp n8n workflow with an Execute SQL node, runs it once, then archives it. Uses existing n8n MCP access.
2. **Dedicated DDL webhook** (full CC autonomy, if provisioned) — a separate webhook accepting arbitrary SQL, protected by API key.
3. **Jordan manual in Supabase SQL Editor** (requires human step) — surface the raw SQL; Jordan pastes at `https://supabase.com/dashboard/project/{{SUPABASE_PROJECT_REF}}/sql`.
4. **psycopg2 direct Postgres** (blocked from Jordan's machine) — port 5432 unreachable from local network; only viable from cloud/VPN context.

If the prompt omits an execution path spec, CC will attempt safe-sql first, hit the block, and must surface SQL to Jordan (option 3 fallback). Option 1 (n8n one-off) is the recommended autonomous path when DDL volume justifies setup overhead.

**safe-sql unusable for reading full JSONB in backfill SELECTs:** Pitfall #31 (safe-sql markdown truncation) applies with full force to MIGRATE backfill tasks. If the migration requires selecting JSONB array columns (e.g., expanding `action_items` into individual rows), safe-sql returns truncated markdown — the JSONB is mid-value cut, unparseable. Use a temp n8n workflow with a native Postgres Execute SQL node for any backfill SELECT that reads JSONB arrays. Archive the temp workflow after use. Do NOT spec "Python + safe-sql" for backfill tasks involving JSONB — safe-sql truncation is guaranteed for multi-element JSONB arrays.

**Field absent from source data — fallback decision required:** When a MIGRATE prompt introduces new columns that should be backfilled from source JSONB (e.g., `newsletter_potential`, `newsletter_angle`), but those fields are absent from all existing rows, CC must make a default-value decision. Prompt should pre-specify the fallback decision explicitly — do not leave it to CC inference mid-execution. Decision table pattern:

| Column | Source field | If absent in source row | Fallback |
|---|---|---|---|
| `newsletter_potential` | `action_items[n].newsletter_potential` | Not present in any of 75 rows | `NULL` |
| `newsletter_angle` | `action_items[n].newsletter_angle` | Not present in any of 75 rows | `NULL` / empty string |

If the prompt omits this table, CC defaults to NULL/empty — which is usually correct but should be an explicit decision, not an implicit one. Add this table to the §MIGRATION mapping section for any MIGRATE task that promotes JSONB sub-fields to top-level columns.
