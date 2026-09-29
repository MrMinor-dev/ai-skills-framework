# Validation Rules

Run these checks before packaging any MRMINOR skill.

---

## Required Checks

### 1. Structure Validation

- [ ] `SKILL.md` exists at skill root
- [ ] YAML frontmatter has `name` and `description`
- [ ] `name` matches folder name (kebab-case)
- [ ] No README.md, CHANGELOG.md, or other clutter

### 2. Token Budget Validation

**Estimate tokens:** Count words × 4, or lines × 4

| Check | Threshold | Action |
|---|---|---|
| SKILL.md body | >1,000 tokens | ❌ FAIL: Split to references |
| Single reference file | >2,000 tokens | ⚠️ WARN: Consider splitting |
| All references combined | >5,000 tokens | ⚠️ WARN: Review necessity |

**Trim priority (if over budget):**
1. Anti-patterns / "don't do" lists
2. Verbose examples
3. Table formatting → inline lists
4. Abbreviations / glossaries
5. Error handling details (keep concise)

### 3. Trigger Validation

**Description must include "Use when:" with specific triggers.**

Check against existing skills for overlap:

| Existing Skill | Triggers |
|---|---|
| `session-startup-skill` | start session, new session |
| `session-end-skill` | end session, wrap up, save progress |
| `semantic-search-skill` | gathering context, finding SSOTs, knowledge retrieval |
| `semantic-index-update-skill` | files changed, after doc edits, stale search results |
| `skill-creator-skill` | creating skills, updating skills |
| `workflow-audit-skill` | audit workflow |
| `workflow-update-skill` | update workflow |
| `workflow-create-skill` | create workflow |
| `doc-audit-skill` | audit docs |
| `doc-management-skill` | create doc, edit doc |
| `strategy-management-skill` | capacity, niches |
| `supabase-operations-skill` | database query |
| `content-compliance-skill` | FTC, Amazon TOS |
| `business-compliance-skill` | deadlines, tax |

**Conflict = same trigger in multiple skills.** Resolve by:
1. Making triggers more specific
2. Consolidating skills
3. Documenting precedence

### 4. Authority Validation

Skill must declare authority tier. Check alignment:

| Skill Actions | Required Tier |
|---|---|
| Read-only, analysis | 3 (autonomous) |
| Doc creation/editing | 2-3 |
| Workflow modifications | 2 |
| External communications | 1 |
| System architecture changes | 1 |
| Financial transactions | 0F (forbidden) |

**Mismatch example:** Skill says "Tier 3" but sends emails → ❌ FAIL

### 5. Dependency Validation

- [ ] All declared dependencies exist
- [ ] No circular dependencies

---

## Validation

Validate manually against this document. The example-skill validator at `/mnt/skills/examples/skill-creator/scripts/quick_validate.py` is optional and Tier-1-gated (Skills Rule 3 — public/example skills require Jordan's approval before use); do not invoke it by default.

---

## Pre-Package Checklist

Before the Step 6 zip:

- [ ] All required checks pass
- [ ] All warnings reviewed with Jordan
- [ ] Skill tested manually in conversation
- [ ] Layer placement confirmed (HAIOS/AOS/PB)
- [ ] Save location identified

---

## Common Issues

| Issue | Solution |
|---|---|
| Description too vague | Add "Use when:" with 3+ specific triggers |
| SKILL.md too long | Move details to references/ |
| Trigger overlap | Make triggers more specific or consolidate |
| Authority mismatch | Align tier with actual actions taken |
