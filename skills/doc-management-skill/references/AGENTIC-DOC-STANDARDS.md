# Agentic Document Standards

Standards for documents that agents can find, understand, and act on autonomously.

---

## Core Principle

Every doc must pass: "Can an agent find this via semantic search, understand it without additional context, and act on it without human clarification?"

---

## Searchability Standards

### Headers as Concepts

Headers become search entry points. They must be meaningful concepts, not generic labels.

| ❌ Bad | ✅ Good |
|---|---|
| Section 1 | Database Schema |
| Overview | Content Strategy Overview |
| Details | Amazon Associates Integration |
| Part A | User Authentication Flow |

### Self-Contained Paragraphs

Semantic search returns chunks (~500 tokens). Each chunk should make sense alone.

**Bad:** "As mentioned above, this connects to the previous system."
**Good:** "The content pipeline connects TikTok uploads to the affiliate tracking system via webhook."

### Keyword Density

Include terms agents will search for. Don't assume context.

**Bad:** "Update the schema when this changes."
**Good:** "Verify schema via live DB query when database tables change: `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '{table}' ORDER BY ordinal_position`"

---

## Reference Standards

### Full Paths Always

Agents can't resolve ambiguous references. Every doc/file reference must be a full path from MRMINOR root.

| ❌ Bad | ✅ Good |
|---|---|
| See the schema doc | Query live DB: `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = '{table}'` |
| Check strategy | Check `AGENCY/Knowledge/STRATEGY-MASTER.md` |
| The workflow spec | `AOS/Workflows/Finance-BudgetMonitor/WORKFLOW-SPEC.md` |

### No Pronouns for Documents

"It" and "this" create ambiguity for agents.

**Bad:** "After updating it, verify the changes."
**Good:** "After updating STRATEGY-MASTER.md, verify cascade dependents."

---

## Structure Standards

### YAML Frontmatter

Every doc starts with machine-readable metadata:

```yaml
---
title: Document Title
version: 1.0
updated: YYYY-MM-DD
purpose: One sentence — what this doc is for
ssot-for: [topic] # Only if this is an SSOT
---
```

### Header Hierarchy

- **H1:** One per doc, matches the concept
- **H2:** Major sections (3-7 per doc ideal)
- **H3:** Subsections as needed
- **H4+:** Rarely — consider splitting doc instead

### Consistent Patterns

Agents learn patterns. Use consistent structures across similar docs:
- All workflow specs use same template
- All SSOTs have same frontmatter fields
- All skills follow SKILL-TEMPLATE.md

---

## SSOT Standards

### Single Location

Each fact/topic lives in exactly ONE place. Duplication = agent confusion.

**Test:** If you search for topic X, do you get exactly one authoritative result?

### Authority Signals

SSOTs use `-MASTER.md` suffix: `STRATEGY-MASTER.md`, `CONTENT-COMPLIANCE-MASTER.md`

### Registry Tracking

All SSOTs listed in `HAIOS/Architecture/SSOT-REGISTRY-MASTER.md` with:
- What it's SSOT for
- Cascade dependents (what to update when this changes)

### Cascade Rules

When SSOT changes, dependents need review. Document these relationships:

```markdown
## Cascade Rules
When this doc changes, check:
- [Dependent doc 1 path]
- [Dependent doc 2 path]
```

---

## Progressive Disclosure

### Lean Core, Detail on Demand

Context windows are finite. Don't front-load everything.

**Pattern:**
- SKILL.md: <500 tokens, workflow only
- references/: Detailed standards, templates, examples
- Agent loads references/ WHEN NEEDED, not upfront

### Chunking-Friendly Length

Individual files should be <2000 tokens when possible. Large files = poor search precision.

**If file exceeds 2000 tokens:** Consider splitting by subtopic.

---

## Verification Checklist

Before any doc is complete:

| Check | Verification |
|---|---|
| Headers are concepts | Read each H2/H3 — is it a searchable term? |
| Full paths used | Search for "see " and "check " — are all refs complete? |
| Self-contained paragraphs | Read random paragraph — does it make sense alone? |
| Frontmatter complete | Has title, version, updated, purpose? |
| SSOT registered | If SSOT, is it in SSOT-REGISTRY-MASTER.md? |
| Cascade documented | If SSOT, are dependents listed? |

**Any "no" = fix before completing.**

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| "See above" / "As mentioned" | Chunks don't include "above" | Repeat key info or use full reference |
| Generic headers | Unsearchable | Use concept nouns |
| Relative paths | Agent can't resolve | Always full path from MRMINOR/ |
| Huge monolithic docs | Poor search precision, context bloat | Split by subtopic |
| Duplicated facts | Conflicting info over time | Single SSOT, reference it elsewhere |
| Missing frontmatter | Can't programmatically parse | Always include YAML header |

---

## Source

Derived from: `HAIOS/Knowledge/BestPractices/KNOWLEDGE-ORGANIZATION-BEST-PRACTICES.md`
