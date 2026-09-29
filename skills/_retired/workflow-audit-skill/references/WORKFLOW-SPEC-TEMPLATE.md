# Workflow Spec Template

Use this template when creating `WORKFLOW-SPEC.md` for audited workflows.

---

```markdown
---
title: [Workflow Name]
workflow-id: [n8n ID]
layer: [HAIOS | AOS | MI]
domain: [Content | Finance | Compliance | Learning | Infra | Email | Database | Comms]
version: 1.0
updated: [YYYY-MM-DD]
session: [N]
status: [operational | testing | disabled | deprecated]
---

# [Workflow Name]

[One sentence: what this workflow does]

## Overview

| Attribute | Value |
|---|---|
| **n8n ID** | `[ID]` |
| **Trigger** | [webhook / schedule / manual] |
| **Schedule** | [cron expression or N/A] |
| **Webhook Path** | [path or N/A] |
| **Active** | [yes / no] |

## Purpose

[2-3 sentences explaining why this workflow exists and what business need it serves]

## Inputs

| Input | Type | Source | Required |
|---|---|---|---|
| [name] | [string/object/etc] | [webhook body / schedule / manual] | [yes/no] |

## Outputs

| Output | Type | Destination |
|---|---|---|
| [name] | [type] | [table / Slack / email / etc] |

## Database

**Tables read:**
- `[table_name]` — [what data]

**Tables written:**
- `[table_name]` — [what data]

## Error Handling

| Scenario | Behavior |
|---|---|
| [error type] | [retry / alert / skip / fail] |

## Dependencies

- **Workflows:** [upstream/downstream workflows]
- **Credentials:** [n8n credential names used]
- **External:** [APIs, services]

## Audit History

| Date | Session | Score | Status |
|---|---|---|---|
| [date] | [N] | [X/65] | [PASS/CONDITIONAL/FAIL] |

## Changelog

| Version | Date | Session | Changes |
|---|---|---|---|
| 1.0 | [date] | [N] | Initial documentation |
```

---

## Naming Convention

**Folder:** `[Layer]/Workflows/[Domain]-[Capability]/`

**Examples:**
- `AOS/Workflows/Finance-BudgetMonitor/WORKFLOW-SPEC.md`
- `AGENCY/Workflows/Content-BlogPipeline/WORKFLOW-SPEC.md`
- `AOS/Workflows/HAIOS-Comms-Slack-Outbound/WORKFLOW-SPEC.md`

## Files in Workflow Folder

```
[Domain]-[Capability]/
├── WORKFLOW-SPEC.md      ← Documentation (required)
├── workflow-backup.json  ← n8n export (recommended)
└── audit-findings.md     ← Latest audit details (if audited)
```
