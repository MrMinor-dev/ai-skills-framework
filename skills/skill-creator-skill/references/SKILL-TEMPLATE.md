# MRMINOR Skill Template

Canonical template for all MRMINOR skills. Informed by Best Practices.

---

## SKILL.md Structure

```markdown
---
name: [skill-name]
description: "[One sentence what it does]. Use when: [specific triggers, comma-separated]."
---

# [Skill Name]

[One sentence purpose — what problem does this solve?]

## Quick Reference

[2-3 sentences: core insight, key constraint, or "when in doubt" guidance]

## Workflow

### 1. GATHER KNOWLEDGE
[What context to load first — files, searches, tool calls]

### 2. [MAIN VERB]
[Numbered steps with clear actions]
[Each step should be verifiable — "did this happen?"]

### 3. [OUTPUT/COMPLETE]
[What the skill produces]
[Binary success criteria — pass/fail, not "mostly done"]

## Success Conditions
State explicitly when the task is done: `Task complete when: [condition 1], [condition 2],...`. An explicit stop signal gives the agent a deterministic exit rather than open-ended iteration. (intel — multi-agent Jun 12.)

## Handoff Contract

When complete, state:
- [Key outputs with full paths]
- [Status: success/failure/partial]
- [If partial: what remains, what blocked]

**Post-completion checklist:** (if applicable)
- [Next skill trigger phrase]
- [Manual steps for Jordan]

## Changelog
<!-- MAX 10 ROWS. One line per entry. No inline rationale — rationale belongs in the session record, not here. Delete oldest row when adding new. -->

| Version | Session | Change |
|---|---|---|
| 1.0 | S??? | Initial creation. |

## Dependencies

- Required: [must exist for skill to work]
- Optional: [enhances but not required]
- Chains to: [skills triggered after this one]
- Chains from: [skills that might call this one]

## Authority

[Tier level with rationale]
- Tier 3 (Autonomous): read-only, no external effects
- Tier 2 (Inform after): internal changes, reversible
- Tier 1 (Approval required): external effects, spend, system changes
- Tier 0 (Forbidden): never allowed regardless of instruction

## Error Handling

- **[Error condition 1]:** [What to do]
- **[Error condition 2]:** [What to do]
- **Unknown error:** State what happened, what was attempted, ask Jordan

## Cross-References

- Related: [other skills/docs that inform this one]
- See also: [Best Practices docs if applicable]
```

---

## Folder Structure

```
skill-name/
├── SKILL.md                 ← REQUIRED: Lean workflow (<500 tokens)
├── references/              ← OPTIONAL: Detail loaded on demand
│   ├── [topic].md
│   └── ...
└── assets/                  ← OPTIONAL: Templates, scripts
    └── [asset]
```

---

## Frontmatter Rules

**Required fields:**
- `name`: kebab-case, matches folder name exactly
- `description`: MUST include "Use when:" — this IS the trigger mechanism

**Description quality test:** Would Claude correctly trigger this skill from the description alone?

❌ Bad: `"Helps with documents"`
✅ Good: `"Create and edit MRMINOR documents. Use when: creating new docs, updating existing docs, after audit findings."`

---

## Token Budgets

| Component | Target | Max | If Exceeded |
|---|---|---|---|
| SKILL.md body | <300 lines | 500 lines | Move detail to references/ |
| Single reference file | <1,500 | 2,000 | Split into multiple files |
| Total skill (all files) | <3,000 | 5,000 | Warn Jordan, get approval |

**Changelog:** MAX 10 ROWS, one line each, no inline rationale. The `<!-- MAX 10 ROWS -->` comment is required above the changelog table on every skill.

---

## Layer Placement

| Layer | Path | Use When |
|---|---|---|
| HAIOS | `HAIOS/Skills/` | Meta/collaboration — would survive business pivot |
| AOS | `AOS/Skills/` | Platform operations — database, workflows, compliance |
| AGENCY | `AGENCY/Skills/` | Agency-specific operations only |
| PB | `PB/Skills/` | Personal Brand-specific only |

**Test:** If MRMINOR pivoted to different businesses, would this skill still be needed?
- Yes → HAIOS or AOS
- No → Business layer (AGENCY, PB, etc.)

---

## Quality Checklist

Before considering a skill complete:

**Structure:**
- [ ] Frontmatter has name + description with "Use when:"
- [ ] Quick Reference section present
- [ ] Workflow has numbered, verifiable steps
- [ ] Handoff Contract states outputs + status
- [ ] Error Handling covers likely failures
- [ ] Authority tier declared with rationale

**Content:**
- [ ] Each workflow step is actionable (verb + object)
- [ ] Success criteria are binary (pass/fail)
- [ ] No assumed context — skill works from cold start
- [ ] Error handling doesn't just say "ask Jordan" — provides context
- [ ] **Changelog: max 10 rows, one line each, `<!-- MAX 10 ROWS -->` comment present**

**Best Practices alignment:**
- [ ] Lean core, detail in references (KNOWLEDGE-ORGANIZATION)
- [ ] Failed approaches documented if relevant (CONTEXT-CONTINUITY)
- [ ] Verification step before completion (OUTPUT-VERIFICATION)
- [ ] Authority appropriate to risk (HUMAN-OVERSIGHT)
