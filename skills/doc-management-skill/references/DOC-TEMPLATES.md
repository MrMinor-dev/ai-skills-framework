# Document Templates

Templates by document type. Copy and customize.

---

## Standard Document

For knowledge docs, guides, references.

```markdown
---
title: [Document Title]
version: 1.0
updated: YYYY-MM-DD
purpose: [One sentence — what this doc is for]
---

# [Document Title]

[Opening paragraph: what this doc covers and why it exists]

## [Section 1 — Concept Name]

[Content organized logically]

### [Subsection if needed]

[Detail]

## [Section 2 — Concept Name]

[Content]

## Cross-References

- Related: [full paths to related docs]
- Depends on: [docs this requires]
- Enables: [docs that depend on this]

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | YYYY-MM-DD | Initial creation |
```

---

## SSOT Document

For Single Source of Truth documents (authoritative reference).

```markdown
---
title: [Topic] Master
version: 1.0
updated: YYYY-MM-DD
purpose: Single source of truth for [topic]
ssot-for: [topic keyword]
---

# [Topic] Master

Authoritative reference for [topic]. All other docs reference this; do not duplicate.

## [Core Content Sections]

[Organized by logical subtopics]

## Cascade Rules

When this doc changes, check:
- [Dependent doc 1 full path]
- [Dependent doc 2 full path]

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | YYYY-MM-DD | Initial creation |
```

**Naming:** Always `[TOPIC]-MASTER.md` (e.g., `STRATEGY-MASTER.md`)

**Registration:** Add to `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md`

---

## Workflow Spec Document

For n8n workflow documentation.

```markdown
---
title: [Workflow Name] Specification
version: 1.0
updated: YYYY-MM-DD
purpose: Specification for workflow [ID]
workflow-id: [n8n workflow ID]
---

# [Workflow Name]

## Overview

**ID:** [workflow ID]
**Trigger:** [what starts it]
**Purpose:** [what it accomplishes]

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| [name] | [type] | Yes/No | [what it is] |

## Outputs

| Output | Type | Description |
|---|---|---|
| [name] | [type] | [what it produces] |

## Flow

```
[Step 1] → [Step 2] → [Step 3]
```

## Error Handling

| Error | Cause | Resolution |
|---|---|---|
| [error] | [why] | [fix] |

## Dependencies

- Requires: [services, credentials, other workflows]
- Called by: [what triggers this]

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | YYYY-MM-DD | Initial creation |
```

**Location:** `Agency/Workflows/[workflow-id]/WORKFLOW-SPEC.md`

---

## Knowledge Document

For research findings, best practices, learnings.

```markdown
---
title: [Topic] Knowledge
version: 1.0
updated: YYYY-MM-DD
purpose: Captured knowledge about [topic]
---

# [Topic]

## Quick Reference

[2-3 sentences: key insight, main takeaway]

## Context

[Why this matters, when it applies]

## Findings

### [Finding 1]

[Detail with source if applicable]

### [Finding 2]

[Detail]

## Implications for MRMINOR

[How this applies to our specific situation]

## Sources

- [Source 1]
- [Source 2]

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | YYYY-MM-DD | Initial creation |
```

---

## Session Capture Document

For session learnings destined for JOURNEY-CAPTURE.md.

```markdown
## Session [N] — [Date]

**Focus:** [What was worked on]

### Decisions Made
- [Decision]: [Rationale]

### Patterns Discovered
- [Pattern]: [When to apply]

### Dead Ends
- [What didn't work]: [Why]

### Open Questions
- [Question for future]
```

**Note:** This is appended to `HAIOS/Journey/JOURNEY-CAPTURE.md`, not a standalone doc.

---

## Domain Document

For AOS domain definitions (YAML spec + implementation status).

**Template location:** `AOS/Templates/DOMAIN-TEMPLATE.md`

**Output location:** `AOS/Domains/[DomainName]/[DOMAIN-NAME]-DOMAIN.md`

**Examples:** `AOS/Domains/Finance/FINANCE-DOMAIN.md`, `AOS/Domains/Email/EMAIL-DOMAIN.md`

---

## Frontmatter Field Reference

| Field | Required | When to Use |
|---|---|---|
| `title` | Always | Human-readable name |
| `version` | Always | Semantic versioning (1.0, 1.1, 2.0) |
| `updated` | Always | Last edit date (YYYY-MM-DD) |
| `purpose` | Always | One sentence explaining doc's reason to exist |
| `ssot-for` | If SSOT | Topic keyword this doc is authoritative for |
| `workflow-id` | If workflow | n8n workflow ID |

---

## Version Numbering

| Change Type | Version Bump | Example |
|---|---|---|
| Typo, minor wording | Patch: x.x.1 | 1.0 → 1.0.1 |
| Content addition, clarification | Minor: x.1.0 | 1.0 → 1.1 |
| Structural change, major rewrite | Major: 2.0.0 | 1.x → 2.0 |
