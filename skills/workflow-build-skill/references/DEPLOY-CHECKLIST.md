# Phase 5: DEPLOY Checklist

**Gate:** All Definition of Done items checked.

## 5.1 Scale Up
Only AFTER Phase 3 gate passed with one input:
- Add remaining inputs (all sources, full data set)
- Test with full volume
- Verify output count matches expectations
- Check for new edge cases at scale (timeouts, rate limits, unexpected data)

## 5.2 Add Error Handling
Add AFTER happy path is proven. Use proven patterns from `BUILD-PATTERNS.md` section 2.6.

**Error Trigger → Postgres (direct INSERT):**
Copy the exact pattern from `BUILD-PATTERNS.md → 2.6 → Error Trigger → Postgres`.
Error Trigger connects directly to a Postgres node. No Code or HTTP nodes in between.
Replace `COMPONENT-NAME` and `WORKFLOW-ID` with actual values.

**Health check on success:**
- End of happy path logs to `haios_health_checks` via Postgres node with `state: 'healthy'`
- Include: component name, layer, items processed (in `raw_result` jsonb), workflow_id

**Node-level resilience:**
- `retryOnFail: true` + `waitBetweenTries: 5000` on Postgres and HTTP nodes
- `onError: "continueErrorOutput"` when Postgres errors should be caught gracefully (see pattern in BUILD-PATTERNS.md)
- `continueOnFail: true` only where partial failure is acceptable (e.g., one subreddit failing shouldn't kill the whole run)

**Credentials reminder:**
After any `n8n_update_full_workflow` call, credentials are dropped. List which nodes need re-credentialing and warn Jordan.

**Health check schema (actual columns):**
```sql
-- haios_health_checks: id (uuid), check_id (varchar), component (varchar),
-- layer (varchar), business (varchar, nullable), timestamp (timestamptz),
-- state (varchar), raw_result (jsonb, nullable), duration_ms (int, nullable),
-- error_message (text, nullable), workflow_id (varchar, nullable)
```

## 5.3 Definition of Done (Build Doctrine Rule 5)

All must be checked before activating:

- [ ] **README updated** — reflects ACTUAL built state, not just the pre-build plan. Update pipeline diagram, node names, any design changes discovered during build.
- [ ] **Domain doc updated** — e.g., `INTELLIGENCE-DOMAIN.md`, `FINANCE-DOMAIN.md`
- **DB schema verified** — if any DB changes, verify via live query: `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '{table}' ORDER BY ordinal_position` (archived — live DB is authoritative)
- [ ] **Error Trigger + health check logging** — both error and success paths log
- [ ] **Workflow activated** OR explicitly marked pending with documented reason
- [ ] **session-context.md updated** — current state, workflow ID, status
- [ ] **MCP warning given** — "MCP access turns OFF after API edits. Re-toggle in n8n."

## 5.4 Activate
```
n8n-mcp:n8n_update_full_workflow → id (set active: true)
```

**Post-activation:**
1. Warn Jordan about MCP access toggle
2. If scheduled workflow → confirm next trigger time makes sense
3. If webhook workflow → test the production webhook URL once

## 5.5 Roadmap Impact (Build Doctrine Rule 4)
After deploying an AOS workflow, ask:
- Does this change what niche #2 onboarding looks like?
- If yes → update `AOS/Architecture/NICHE-SCALING-PLAYBOOK.md` or `AOS/Domains/SERVICE-CATALOG.md`

## Phase 5 Complete When:
- [ ] Full volume tested
- [ ] Error handling in place and tested (trigger an error intentionally if safe to do so)
- [ ] All DoD items checked
- [ ] Workflow active (or pending with reason)
- [ ] Docs updated: README, domain doc, schema doc, session-context
- [ ] Jordan warned about MCP toggle
