# Feedback Classification Table

Maps signals in feedback files to target files and authority tiers.

| Signal in feedback | Target file | Tier |
|---|---|---|
| Pattern failure, repeated mistake, "anti-pattern" | `HAIOS/Knowledge/BestPractices/ANTI-PATTERNS.md` | 3 |
| Any completed task cycle worth logging | `AOS/Skills/cc-prompt-skill/references/FEEDBACK-LOG.md` | 3 |
| n8n API behavior, undocumented field, platform quirk | `AOS/Domains/Infra/N8N-API-REFERENCE.md` | 3 |
| Global convention gap / behavior rule | `~/.claude/CLAUDE.md` | 1 |
| Prompt template gap or structural improvement | `AOS/Skills/cc-prompt-skill/references/PROMPT-TEMPLATE.md` | 2 |
| New persistent behavioral rule needed | `~/.claude/rules/{name}.md` | 1 |
| Skill update needed | `~/.claude/skills/{name}/SKILL.md` | 1 |
| Hook addition or change | `.claude/settings.json` / hook script | 1 |

## Tier Definitions

| Tier | Name | Meaning | How to handle |
|---|---|---|---|
| 3 | Autonomous | Low-risk reference data (anti-patterns, feedback log, API docs) | Apply immediately. Log in report. |
| 2 | Inform After | Convention-level changes (CLAUDE.md, prompt template) | Apply immediately. Log in report. Flag for COO review. |
| 1 | Approval Required | Behavioral/structural changes (rules, skills, hooks) | Draft only — do NOT apply. COO decides. |

> **Note:** CLAUDE.md is Tier 1 (draft-only) — every-message tax, never auto-applied. Cross-ref: CC-CONFIG-SELF-IMPROVEMENT-ARCHITECTURE.md §5 + OPEN-6.
