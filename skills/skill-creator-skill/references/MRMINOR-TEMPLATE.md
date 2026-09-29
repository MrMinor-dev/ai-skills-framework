# MRMINOR Skill Template

Use this template when creating new skills. Copy and customize.

---

## SKILL.md Template

```markdown
---
name: [skill-name]
description: "[What it does]. Use when: [specific triggers]."
---

# [Skill Name]

[One sentence purpose]

## Workflow

### 1. GATHER KNOWLEDGE
- [What context to load first]

### 2. [MAIN STEPS]
[Numbered steps with clear actions]

### 3. [OUTPUT/COMPLETION]
[What the skill produces]

## Handoff Contract

When complete, state:
- [Key outputs with paths]
- [Status/result]

## Dependencies

- Required: [skills/tools that must exist]
- Optional: [skills that enhance but aren't required]

## Authority

[Tier level per Operating Agreement]
- Tier 3: Autonomous
- Tier 2: Inform after
- Tier 1: Approval required

## Error Handling

[What to do when things go wrong]
```

---

## Folder Structure

```
skill-name/
├── SKILL.md                 ← REQUIRED: Lean workflow (<500 lines, <1k tokens)
├── references/              ← OPTIONAL: Detailed docs loaded on demand
│   ├── [topic].md          ← Split by topic for progressive loading
│   └── ...
├── scripts/                 ← OPTIONAL: Executable code
│   └── [script].py
└── assets/                  ← OPTIONAL: Templates, images for output
    └── [asset]
```

---

## Frontmatter Rules

**Required fields:**
- `name`: kebab-case, matches folder name
- `description`: MUST include "Use when:" with specific triggers

**Description is THE trigger mechanism.** Claude reads only name+description to decide which skill to load. Everything in body loads AFTER triggering.

Bad: `description: "Helps with documents"`
Good: `description: "Create and edit MRMINOR documents. Use when: creating new docs, updating existing docs, after audit findings require doc changes."`

---

## Token Budget Targets

| Component | Target | Max | Action if Exceeded |
|---|---|---|---|
| SKILL.md body | <500 tokens | 1,000 | Split to references/ |
| Single reference | <1,500 tokens | 2,000 | Split further |
| All references | <3,000 tokens | 5,000 | Warn Jordan |

**Estimation:** ~4 tokens per word, ~100 tokens per 25 lines

---

## Layer Placement Guide

| Layer | Location | When to Use |
|---|---|---|
| HAIOS | `HAIOS/Skills/[skill-name]/` | Meta/collaboration (sessions, knowledge, docs) |
| AOS | `AOS/Skills/[skill-name]/` | Platform operations (database, workflows, compliance) |
| PB | `PB/Skills/[skill-name]/` | Personal Brand-specific only |

**Test:** Would this skill still be needed if MRMINOR pivoted to different businesses?
- Yes → HAIOS or AOS
- No → Business instance level (a former e-commerce project/PB)

---

## Orchestration Pattern

Skills are orchestrated by Claude (or Jordan), not chained automatically.

**Handoff Contract pattern:**
```markdown
## Handoff Contract
When complete, state:
- Files created/modified: [list with full paths]
- Status: [result]
```

Claude decides what to do next based on context and Jordan's direction.

---

## Examples by Type

**Knowledge skill** (reads, synthesizes):
- Primary tools: semantic search, file read
- Low authority (Tier 3)
- Example: `session-startup-skill`

**Workflow skill** (executes n8n):
- Primary tools: n8n MCP
- Medium authority (Tier 2)
- Example: `workflow-audit-skill`

**Doc skill** (creates, edits files):
- Primary tools: Desktop Commander
- Medium authority (Tier 2-3)
- Example: `doc-management-skill`

**Utility skill** (internal tool for Claude):
- Not user-triggered, used during other work
- Low authority (Tier 3)
- Example: `semantic-search-skill`
