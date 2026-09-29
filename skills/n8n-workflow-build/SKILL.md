---
name: n8n-workflow-build
description: "Build or modify n8n workflows from a COO prompt spec via REST API. Activates when task involves creating, updating, or fixing n8n workflows in AOS."
---

# n8n Workflow Build

Execute workflow builds from COO-provided prompt specs using the n8n REST API.

## Context

CC receives a fully-specced prompt file from COO. The spec includes: node-by-node configuration, connections, test payloads with expected results, and audit criteria. CC's job is execution — build, test, audit, deliver.

**API reference:** @AOS\Domains\Infra\N8N-API-REFERENCE.md
**Node configs:** @AOS\Domains\Infra\N8N-NODE-CONFIGS.md
**Anti-patterns:** Filter `[n8n]` entries from `HAIOS\Knowledge\BestPractices\ANTI-PATTERNS.md` (canonical master). `references/n8n-anti-patterns.md` was deprecated and archived to `~/.claude/_archive-cruft/skills-n8n-workflow-build-references/` (canonical remains `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md`).
**API patterns:** @references/n8n-api-patterns.md

## The Full Loop

```
1. Orchestrator spawns BUILDER agent with prompt spec
2. Builder: access check → build → self-test → report
3. Orchestrator spawns AUDITOR agent (fresh context) with workflow ID
4. Auditor: score against 60-pt checklist → return JSON
5. Orchestrator evaluates:
   PASS (≥48/65, no blockers) → post-build flow
   CONDITIONAL (36-47)       → send findings to builder, re-test, re-audit
   FAIL (<36)                → escalate to Jordan with full trail
6. Max 3 build→audit cycles. After 3 failures → escalate.
```

## Build Steps (for Builder agent)

### 1. ACCESS CHECK
Before any work, verify:
- n8n API key readable from `.secrets/n8n-api-key.txt`
- Supabase access via safe-sql (if DB verification needed)
- All files referenced in prompt are readable

### 2. BUILD
- Read existing workflow (if modifying): `GET /api/v1/workflows/{id}`
- Construct or modify nodes + connections per spec
- Strip rejected API fields (see n8n-workflows-core rule)
- Write workflow: `PUT /api/v1/workflows/{id}` or `POST /api/v1/workflows`
- Validate by reading it back and comparing against spec

### 3. SELF-TEST
- Activate workflow
- Send each test payload from the prompt's test table
- For each test: check execution results against expected output
- Log what each test CAN and CANNOT verify (webhook limitations)
- Deactivate workflow

### 4. FIX (if tests reveal issues)
- One fix per cycle. Re-test after each fix.
- Max 3 fix cycles before reporting. Read execution data to diagnose — never guess.

### 5. REPORT TO ORCHESTRATOR
Builder reports status. Orchestrator decides whether to proceed to audit.

### 6. POST-BUILD: BACKUP & TAG
After verification passes:
1. GET the final workflow JSON from n8n API
2. Save to `{workflow-folder}/workflow.json` (pretty-printed)
3. Assess remediation maturity level (L0-L4) based on health check nodes, Error Trigger, and remediation webhook connection
4. If workflow name doesn't have accurate [L#] tag, update via PUT
5. If workflow was renamed during build, update README Name History table

## Post-Audit Flow (for Orchestrator)

When the auditor returns PASS (≥48/65, zero blockers):

### Activation Gate
- Workflow is NOT auto-activated. Jordan approves activation.
- Orchestrator presents to Jordan:
  ```
  AUDIT PASSED: {workflow name} ({id})
  Score: {score}/65
  
  Jordan action items:
  - [ ] Review audit score and findings
  - [ ] {each item from coo_action_items}
  - [ ] Approve activation (or flag concerns)
  
  CC will activate once you confirm.
  ```

### Doc Updates (CC handles)
- README updated with final BUILD STATE (reflects actual, not planned)
- Domain doc updated (e.g., `INTELLIGENCE-DOMAIN.md`)
- Schema doc updated if any DB changes (`SUPABASE-SCHEMA-MASTER.md`)
- `session-context.md` updated with workflow status
- Reindex changed files via `semantic-reindex` skill

### Feedback File
- CC writes feedback file per the prompt's Feedback Protocol
- Feedback stays in place for COO review (prompt file archived)

## Rules

### Knowledge Classification (AOS-first)
This skill's `references/` files contain **universal n8n platform knowledge** — patterns and anti-patterns that apply to ANY workflow regardless of business, niche, or integration. Integration-specific patterns (Xpoz, Stripe, Twilio, etc.) belong in the **workflow README**, loaded via the prompt's CONTEXT section. An integration graduates to universal only when 5+ workflows depend on it.

- **Universal (goes here):** API mutation rules, expression parser gotchas, naming conventions, PUT body spec, health check patterns
- **Integration-specific (goes in README):** Xpoz SSE format, third-party API auth workarounds, service-specific response parsing
- **Business-specific (goes in prompt):** Source lists, table names, schedule times, credential IDs

### Naming Convention
- **Workflow name format:** `[Layer]: [Domain] - [Workflow Name] [Lx]` (e.g., `HAIOS: Comms - Daily Digest [L2]`, `AOS: Finance - Invoice Handler [L2]`, `Agency: Blog - Publisher [L2]`)
- **Layer values:** `HAIOS`, `AOS`, `Agency`, or venture name (no square brackets around layer)
- **Version:** NOT in the workflow name. Version history lives in the README `## Changelog` table only.
- **Drive folder name:** Sanitized n8n name — replace `:` with `-` (e.g., `HAIOS - Comms - Daily Digest [L2]`). **Never use workflow ID as folder name.**
- **Node names:** Descriptive, unique within workflow. Verb-first for actions (e.g., `Query Twitter Sources`, `Upsert Blog Signals`). Noun-first for data holders (e.g., `Twitter Batch Loop`).
- **Workflow IDs in docs:** Always record in README frontmatter after creation.

### Build Rules
- Never create nodes from memory. The prompt spec defines every node.
- **For standard nodes (Webhook, Code, HTTP Request, IF, Switch, Merge, Set, Postgres, Error Trigger, Schedule Trigger): use JSON patterns from `N8N-NODE-CONFIGS.md`. Do NOT call `get_node_types` for these.** Only call `get_node_types` for nodes not in that file. Use `validate_workflow` to catch errors iteratively.
- Preserve all existing node IDs when modifying workflows.
- Credential-bearing nodes: pass credential objects unchanged from GET response.
- After any REST API edit, warn Jordan about MCP access toggle.
- Human-in-the-loop for activation — CC never activates without Jordan's explicit approval.
