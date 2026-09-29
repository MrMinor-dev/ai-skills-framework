---
name: workflow-audit-skill
description: "Audit n8n workflows. Triggers: 'audit workflow', 'check workflow', 'review workflow'."
---

# Workflow Audit [DEPRECATED]

**Version:** 2.1 | **DEPRECATED:**

> **DEPRECATED.** This COO skill is retired. COO no longer performs hands-on workflow auditing -- CC handles all n8n operations. Verification is now split: CC follows `.claude\rules\workflow-verification.md` (L1-L3), COO performs independent L4 check via safe-sql per cc-prompt-skill v1.4. Remove this skill from Claude skills settings.

Two-phase audit: JSON analysis (33 pts) → Runtime verification (27 pts). Total 60 pts.

## Quick Reference

| Phase | Points | Scope | Tool |
|---|---|---|---|
| 1: JSON | 33 | Parse workflow structure | `n8n:get_workflow_details` |
| 2: Runtime | 27 | External verification | Safe SQL, executions, files |

**Thresholds:** PASS ≥48 (80%) | CONDITIONAL 36-47 | FAIL <36

---

## Phase 1: JSON Analysis (33 pts)

### 1.1 FETCH
```
n8n:get_workflow_details → workflowId
```
Captures: name, ID, active, trigger, nodes, connections.

### 1.2 CLASSIFY

| Layer | Criteria |
|---|---|
| HAIOS | Jordan+Claude collaboration |
| AOS | LLC-wide operations |
| Agency | Business-specific |

### 1.3 SCORE (JSON-Observable)

| Cat | Pts | What to Parse |
|---|---|---|
| A: Triggers | 6 | Trigger node type, count, path/schedule |
| D: Strategy | 3 | Purpose aligns with domain |
| G: Metrics | 4 | Logging nodes present |
| I: Tech Stack | 3 | Node types, versions, credentials |
| J: Cadence | 5 | Schedule expression matches purpose |
| L: Parameters | 5 | No hardcoded secrets in Set/Code nodes |
| M: Error Handling | 7 | `onError`, `retryOnFail`, logs to `haios_health_checks` |

**Error Handling Pattern:**
- All workflows: Error Trigger → Log to `haios_health_checks` (state='error')
- Daily Digest surfaces errors from health checks — no Slack needed for standard errors
- Slack alerts (`#haios-alerts`) ONLY for critical/time-sensitive failures (e.g., compliance deadlines)

See `references/AUDIT-CHECKLIST.md` for detailed criteria.

### 1.4 PHASE 1 OUTPUT

```markdown
## Phase 1: JSON Analysis — {workflow name}
**ID:** {id} | **Session:** {N}
**Layer:** {layer} | **Domain:** {domain}

### Scores (33 pts possible)
| Cat | Score | Evidence |
|---|---|---|
| A | X/6 | {finding} |
| D | X/3 | {finding} |
| G | X/4 | {finding} |
| I | X/3 | {finding} |
| J | X/5 | {finding} |
| L | X/5 | {finding} |
| M | X/7 | {finding} |
| **Total** | **X/33** |  |

### Critical Issues (Phase 1)
- {issues that block runtime testing}

### Ready for Phase 2: {YES/NO}
```

**Checkpoint:** Present Phase 1 output. Ask: "Proceed to Phase 2?"

---

## Phase 2: Runtime Verification (27 pts)

Requires Phase 1 complete. Uses external tools.

### 2.1 VERIFY (Runtime Checks)

| Cat | Pts | Tool | What to Check |
|---|---|---|---|
| B: Outputs | 5 | `n8n:list_executions` | Last 10 runs, success rate |
| C: Database | 4 | Safe SQL | Tables exist, recent records |
| E: Testing | 4 | `n8n:execute_workflow` | Safe test (if webhook) |
| F: Docs | 4 | Desktop Commander | README.md exists in workflow folder |
| H: COO Access | 5 | Verify | Webhook callable, outputs queryable |
| K: Indexed | 5 | Semantic search | Doc findable |

### 2.2 DOCUMENT

Create/update: `[Layer]/Workflows/[Domain]-[Capability]/README.md`

### 2.3 FINAL REPORT

```markdown
## Workflow Audit: {name}
**ID:** {id} | **Session:** {N} | **Date:** {date}
**Layer:** {layer} | **Domain:** {domain}
**Score:** {X}/60 ({%}) | **Status:** {PASS/CONDITIONAL/FAIL}

### Phase 1: JSON Analysis (X/33)
| Cat | Score | Evidence |
|---|---|---|
| A | X/6 | {finding} |
...

### Phase 2: Runtime (X/27)
| Cat | Score | Evidence |
|---|---|---|
| B | X/5 | {finding} |
...

### Findings
- **Critical:** {must fix}
- **Recommended:** {should fix}

### Actions
| Action | Owner | Status |
|---|---|---|
| {action} | {owner} | 📋 |
```

---

## Handoff Contract

**Phase 1 complete:**
- Workflow: [name] ([ID])
- Phase 1 score: X/33
- Ready for Phase 2: YES/NO

**Full audit complete:**
- Final score: X/60 (PASS/CONDITIONAL/FAIL)
- Files created: [paths]
- Critical issues: [count]
- Rename needed: [old] → [new]

## Dependencies

- n8n MCP: `get_workflow_details`, `list_executions`, `execute_workflow`
- Desktop Commander: file operations
- Safe SQL: `{{WORKFLOW_ID}}` (webhook)
- Semantic search: `{{WORKFLOW_ID}}` (webhook)

## Authority

Tier 3 — read-only operations
Tier 1 — test execution (requires approval)

## Error Handling

- **Workflow not found:** Check ID, may be in different project
- **No executions:** Score B=0, flag as untested
- **MCP issue:** Document what couldn't be verified
