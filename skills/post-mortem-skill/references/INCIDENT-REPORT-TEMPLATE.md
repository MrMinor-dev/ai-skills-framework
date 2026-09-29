---
title: INCIDENT-REPORT-TEMPLATE
purpose: Standard template for all MRMINOR incident post-mortems
version: 1.0
created: 2026-06-01 (PM-7 cascade; sourced from I20260601-1 exemplar)
usage: Copy → rename to I{YYYYMMDD}-{N}-{slug}.md → fill sections per severity
---

# [INCIDENT TITLE] Post-Mortem — [Brief description]

## Frontmatter (fill and move to top of file)

```
---
title: [title]
incident_id: I{YYYYMMDD}-{N}
session: S{N}
date: YYYY-MM-DD
severity: P0 | P1 | P2
status: OPEN | CLOSED
review-state: draft | approved
ssot-for: [slug]-retrospective
layer: HAIOS
related:
  - [related docs]
---
```

---

## 1. Executive Summary

One paragraph: what happened, when, what the impact was, and how it was resolved. Include the lineage if this is a descendant of a prior incident.

---

## 2. Timeline

| Time | Event |
|---|---|
| T-0 | Incident onset (actual or estimated) |
| T+X | Detection |
| T+X | Mitigation started |
| T+X | Resolved |

Detection lag = T+Detection - T-0. Any gap > 30 min for a P0/P1 is a secondary finding.

---

## 3. Root Cause Analysis

**Why it happened (3-Why for P1/P2, 5-Why for P0):**

1. Why did [symptom] occur? → Because [mechanism]
2. Why did [mechanism] occur? → Because [upstream cause]
3. Why did [upstream cause] exist? → Because [structural gap]
(P0: continue to 5)

**Root cause (one sentence):** _____

**Contributing factors:**
- [Factor 1]
- [Factor 2]

---

## 4. Impact

| Dimension | Impact |
|---|---|
| Workflows affected | [list] |
| Executions burned | [count] |
| Duration | [start → end] |
| Data integrity | [any writes corrupted?] |
| Revenue / brand | [if applicable] |

---

## 5. Corrective Actions

| # | Action | Tier | Status |
|---|---|---|---|
| C1 | [action] | 0H / 1 / 2 / 3 | OPEN / DONE S{N} |
| C2 | [action] |  |  |

---

## 6. AP Candidates

Per Quick Rule #13: grep ANTI-PATTERNS.md for `| {N} |` before recording any AP number.

| Pattern | n (occurrences) | Disposition |
|---|---|---|
| [Pattern description] | [count] | PROMOTE to anti-pattern {N} / Watch (W) / Novel |

---

## 7. Best Practices Cascade

For each second-order impact found:

```
[Area]: [What the incident reveals] -> [Specific doc/rule/skill] -> [Tier 3=now / Tier 2=queue / Tier 1=Jordan]
```

Areas to scan: Doctrines, Architecture docs, Build templates, CC rules, Build roadmaps, Skills, SSOT registry, Ledger/budgets.

If cascade > 5 items → create `HAIOS/Sonnet-Cascade/S{N}-{slug}-cascade.md`.

---

## 8. Ledger + Runbook Updates

- EXECUTION-BUDGET-LEDGER: [row updated? which?]
- KILL-SWITCH-RUNBOOK: [current? updated?]
- CREDENTIAL-MAP: [row flagged? updated?]

---

## Exemplar Reference

This template is sourced from `HAIOS/Post-Mortems/I20260601-1-stale-credential-lineage-close.md` (Stale-Credential Lineage Retrospective). Refer to that doc for a full worked example of a P1 multi-mechanism post-mortem with AP candidates and cascade.
