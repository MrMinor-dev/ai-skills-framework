---
name: post-mortem-skill
description: "Write incident post-mortems and close the learning loop. Use when: post-mortem, incident report, write up incident, kill-switch fired, budget alert fired, outage report, n8n incident."
updated: 2026-06-01 (Drive canonical home created PM-7 cascade)
drive-canonical: true
version: 1.1
---

# Post-Mortem Skill

**Version:** 1.1 | **Layer:** AOS

Forcing function for turning every operational incident into a learning. Without this skill, incidents get fixed and forgotten — the kill-switch only compounds if fires generate entries.

## Quick Reference

```
Incident detected → classify severity → fill template → persist to Drive → corrective actions → AP candidates → best practices cascade → ledger + runbook updates → announce
```

Default output: `HAIOS/Post-Mortems/I{YYYYMMDD}-{N}-{slug}.md`

Template: `AOS/Skills/post-mortem-skill/references/INCIDENT-REPORT-TEMPLATE.md`

---

## Severity Classification

| Level | Trigger | Example | Template depth |
|---|---|---|---|
| **P0** | Full outage, kill-switch fired, site down | cascade | Full 5-Why + corrective actions + full cascade review |
| **P1** | Degraded operation, Budget Monitor 90%+ | Single workflow burn | 3-Why + corrective actions + cascade review |
| **P2** | Alert fired, investigated, no outage | Budget Monitor 75% cleared quickly | Summary + root cause + quick cascade scan |

When uncertain, default to **P1**.

---

## Workflow

### 1. GATHER INCIDENT DATA

Pull the facts before writing. Do not reconstruct from memory.

- **Detection:** What fired the alert? Budget Monitor / Kill-switch Slack / Jordan noticed / UptimeRobot
- **Timestamp:** When did it start? When was it detected? Gap = detection lag
- **Scope:** Which workflows were affected? How many executions burned?
- **Resolution:** What was the fix? How long did recovery take?

For n8n incidents: GET /api/v1/executions?limit=250&startedAfter={onset-timestamp} — pull execution breakdown by workflow.

### 2. CLASSIFY AND SLUG

- Severity: P0 / P1 / P2
- Slug: 3-5 words, kebab-case (e.g., intake-processor-retry-storm)
- ID: I{YYYYMMDD}-{N} where N = 1-indexed count of incidents that day

### 3. FILL TEMPLATE

Read `AOS/Skills/post-mortem-skill/references/INCIDENT-REPORT-TEMPLATE.md`. Fill all sections. Depth scales with severity:
- P0: all sections including 5-Why + full corrective action table + full cascade review
- P1: 3-Why + corrective actions + cascade review
- P2: Summary + root cause + quick cascade scan (skip corrective actions if already resolved)

### 4. PERSIST TO DRIVE

Persist-Before-Present rule applies. Write to Drive FIRST.

  Filesystem:write_file to HAIOS\Post-Mortems\I{YYYYMMDD}-{N}-{slug}.md

Status: OPEN until corrective actions complete. Set CLOSED when all Tier 2/3 actions done or explicitly deferred.

### 5. CORRECTIVE ACTIONS

For each action identified, tier it:

| Tier | Scope | Examples |
|---|---|---|
| 0H | Jordan only | Credential rotation, billing changes |
| 1 | Approval required | Architecture changes, new infrastructure |
| 2 | Inform after | Workflow cadence changes, doc updates |
| 3 | Autonomous | AP entries, LEDGER updates, runbook edits |

Execute Tier 3 in-session. Queue Tier 2 for session-end. Tier 1 surfaces to Jordan. Tier 0H = Jordan action item.

### 6. AP CANDIDATES

For each pattern observed:
- Count prior occurrences (n). Check HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md.
- If n >= 2: promote to AP entry now (Tier 3)
- If n = 1: add to Watch list (W##)
- If novel: note in post-mortem only; watch for recurrence

Per Quick Rule #13: grep ANTI-PATTERNS.md for | {N} | before recording any AP number.

### 7. BEST PRACTICES CASCADE

The highest-leverage step. Ask: what does this incident reveal about our systems, principles, and rules that isn't yet codified?

Scan these areas:

| Area | Ask |
|---|---|
| Doctrines + principles | Does this contradict or extend an existing principle? Does a new one need writing? |
| Architecture docs | Does the failure mode reveal a structural gap in SITE-RELIABILITY-ARCHITECTURE.md, AUTONOMOUS-EXECUTION-ARCHITECTURE.md, or a domain SSOT? |
| Build templates | Does this reveal something BUILD prompts should enforce at authoring time? |
| CC rules | Should a rule fire automatically in CC sessions to prevent recurrence? |
| Build roadmaps | Does this open a new workstream or reprioritize an existing one? |
| Skills | Does a skill need a new step, gate, or forcing function? |
| SSOT registry | Are new docs born from this incident that need registration in SSOT-REGISTRY-MASTER.md? |
| Ledger/budgets | Flag here if broader cadence policy changes are needed (detail in Step 8). |

Format: `[Area]: [What the incident reveals] -> [Specific doc/rule/skill to update] -> [Tier]`

If cascade is large (>5 items), create a named cascade file at `HAIOS/Sonnet-Cascade/`.

### 8. LEDGER + RUNBOOK UPDATES

- If cadence was root cause: update AOS/Domains/Infra/EXECUTION-BUDGET-LEDGER.md row
- If kill-switch fired: verify AOS/Domains/Infra/KILL-SWITCH-RUNBOOK.md is current
- If credential was root cause: flag AOS/Domains/Infra/CREDENTIAL-MAP.md row

### 9. ANNOUNCE

State to Jordan:
- Post-mortem path
- Severity: P{N}
- Status: OPEN | CLOSED
- Tier 1/0H actions requiring Jordan attention (explicit list)
- Any new AP entries written
- Best practices cascade items executed + queued

---

## Handoff Contract

After completing a post-mortem:
- File persisted to Drive
- Tier 3 actions executed (APs, LEDGER, runbook, cascade items)
- Tier 2 items in session-context for session-end
- Jordan informed of Tier 1/0H items
- Cascade review complete (even if result is "no second-order impact found")

---

## Dependencies

- Filesystem tools (write_file, read_text_file)
- `AOS/Skills/post-mortem-skill/references/INCIDENT-REPORT-TEMPLATE.md`
- AOS/Domains/Infra/EXECUTION-BUDGET-LEDGER.md
- AOS/Domains/Infra/KILL-SWITCH-RUNBOOK.md
- HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md

---

## Authority

- Writing post-mortem + Tier 3 actions + cascade execution: Tier 3 (autonomous)
- Tier 2 actions: Tier 2 (inform after)
- Tier 1/0H: surfaces to Jordan

---

## Changelog

| Version | Changes |
|---|---|
| 1.1 | Drive canonical home created (PM-7 cascade). Template path updated to `AOS/Skills/post-mortem-skill/references/INCIDENT-REPORT-TEMPLATE.md`. |
| 1.1 | Step 7 (Best Practices Cascade) added. |
| 1.0 | Created. established the pattern. |
