# AI Skills Framework

**25 versioned skill files that make an AI agent run the same procedure the same way every time. The real files are here.**

Ask an AI to "update the session context" and you get a different answer each time. Different fields, different format, different scope. Nothing is exactly wrong. But inconsistency in a production system is the same as unreliability.

The usual fix is a longer system prompt. That holds for simple tasks. It falls apart at 10 complex capabilities, each with its own inputs, outputs, and edge cases. A prompt long enough to cover all of them makes everything worse, because the model attends to thousands of tokens of instructions when it needs one.

Maintenance is the other half. When a skill has to change, where does the change go? In a system prompt, you are editing a monolith. If it isn't written down anywhere, the change doesn't stick, and the next session goes back to default behavior.

So each capability became a skill: one Markdown file with a version number, read on demand.

## What's here

**Sessions**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [session-end-skill](skills/session-end-skill/SKILL.md) | 1.23 | Chat assistant | Closes a session: records decisions, updates the state document, flags unfinished work. |
| [session-start-day-skill](skills/session-start-day-skill/SKILL.md) | - | Chat assistant | Gives the daily briefing on the first session of the day. |
| [session-startup-skill](skills/session-startup-skill/SKILL.md) | - | Chat assistant | Reloads state for a mid-day session and routes to a focus area. |

**Knowledge**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [doc-management-skill](skills/doc-management-skill/SKILL.md) | 2.23 | Chat assistant | Creates and edits documents to a fixed standard: frontmatter, source-of-truth status, registry entry. |
| [quarterly-doc-audit-skill](skills/quarterly-doc-audit-skill/SKILL.md) | 2.5 | Chat assistant | Runs the quarterly documentation audit in phases. |
| [semantic-index-update-skill](skills/semantic-index-update-skill/SKILL.md) | - | Chat assistant | Updates the search index after files change. |
| [semantic-reindex](skills/semantic-reindex/SKILL.md) | - | Coding agent | Incrementally updates the search index for changed files. |
| [semantic-search](skills/semantic-search/SKILL.md) | - | Coding agent | Searches the knowledge base for context (Claude Code version). |
| [semantic-search-skill](skills/semantic-search-skill/SKILL.md) | - | Chat assistant | Searches the knowledge base before any large file is loaded. |
| [wikify-skill](skills/wikify-skill/SKILL.md) | 1.0 | Chat assistant | Turns raw assets into wiki pages in 4 steps: extract, route, generate, validate. |

**Workflow engineering**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [n8n-diagnose](skills/n8n-diagnose/SKILL.md) | - | Coding agent | Diagnoses failing n8n workflows in 4 phases: evidence, diagnose, fix, verify. |
| [n8n-workflow-audit](skills/n8n-workflow-audit/SKILL.md) | - | Coding agent | Scores an n8n workflow against the 65-point checklist. |
| [n8n-workflow-build](skills/n8n-workflow-build/SKILL.md) | - | Coding agent | Builds or modifies n8n workflows from a spec through the REST API. |
| [workflow-build-skill](skills/workflow-build-skill/SKILL.md) | 2.0 | Chat assistant | Builds n8n workflows from a spec, with the audit as a gate. |
| [workflow-reactivation-skill](skills/workflow-reactivation-skill/SKILL.md) | 1.8 | Chat assistant | Checklist for moving a workflow out of production or back into it. |

**Data**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [database-skill](skills/database-skill/SKILL.md) | - | Chat assistant | Runs database operations safely: checks the live schema first, then works through the guarded services. |

**Oversight**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [archive](skills/archive/SKILL.md) | - | Coding agent | Moves finished artifacts to a cold archive under self-locating names. |
| [cc-audit-skill](skills/cc-audit-skill/SKILL.md) | 1.0 | Chat assistant | Audits a finished Claude Code run and classifies it clean, issue or storm. |
| [post-mortem-skill](skills/post-mortem-skill/SKILL.md) | 1.1 | Chat assistant | Writes incident post-mortems and closes the learning loop. |
| [process-feedback](skills/process-feedback/SKILL.md) | 1.9 | Coding agent | Turns a Claude Code feedback file into proposed environment updates. It proposes, and a person approves. |

**Delegation and authoring**

| Skill | Version | Runs in | What it does |
|---|---|---|---|
| [cc-prompt-skill](skills/cc-prompt-skill/SKILL.md) | 1.38 | Chat assistant | Writes the task prompts that delegate work to Claude Code, and processes the feedback that comes back. |
| [github-update-skill](skills/github-update-skill/SKILL.md) | - | Coding agent | Drafts GitHub READMEs and the prompts that push them. |
| [pdf-creation-skill](skills/pdf-creation-skill/SKILL.md) | 1.0 | Chat assistant | Builds PDF documents. |
| [pptx-creation-skill](skills/pptx-creation-skill/SKILL.md) | 1.0 | Chat assistant | Builds presentation decks. |
| [skill-creator-skill](skills/skill-creator-skill/SKILL.md) | 3.8 | Chat assistant | Creates, validates and packages new skills. |

**Retired**

| Skill | Version | What it was |
|---|---|---|
| [workflow-audit-skill](skills/_retired/workflow-audit-skill/SKILL.md) | 2.1 | The original 60-point workflow audit. Replaced by n8n-workflow-audit. |

| Other files | What it is |
|---|---|
| [LICENSE](LICENSE) | MIT. |

Skills are sanitized. Internal paths, IDs, session references and similar are removed or replaced with placeholders, and the file text is otherwise as it runs. Some skills name my working setup: a chat assistant I call the COO, the coding agent, Google Drive folders. The [glossary](https://github.com/MrMinor-dev/ai-operating-system/blob/main/GLOSSARY.md) explains the labels.

`skills/_retired/` holds a skill I retired. It stays as a record of how the workflow audit grew from 60 points to 65.

I keep other skills private. They cover personal writing and sales work.

## How a skill is built

A skill is a folder with a `SKILL.md`. The file is the contract.

```
skill-name/
  SKILL.md          frontmatter (name, description with trigger phrases),
                    then a short workflow the agent follows step by step
  references/       optional detail, loaded only when the step needs it
```

The rules `skill-creator-skill` checks before a skill ships:

- The description has trigger phrases, and they don't overlap with any other skill's triggers.
- The body stays short enough to read under context pressure. Detail moves to `references/`.
- The authority level is declared, using the [tiers from the governance framework](https://github.com/MrMinor-dev/security-governance-framework).
- The changelog stays short, one line per version.

**Invocation.** The agent sees a trigger phrase, reads the skill file, and works from the file. It never works from memory. The read is mandatory.

## Why the read is mandatory

Instructions loaded at the start of a long session get less attention than instructions loaded recently. A skill run late in a long session can behave differently than the same skill run early, if it leans on memory. A fresh read resets that.

The second reason is versioning. `session-end-skill` version 1.1 overwrote the session history file when it should have appended. It destroyed months of accumulated organizational history. Version 1.2 fixed the bug. The fix only took hold because every run reads the current file. Had the agent been running from cached behavior, the bug would still be live. The version bump is the audit trail.

This pattern has a name: skill drift. The skill's behavior slowly moves away from its spec because the model fills gaps from memory and from what usually happens. The mandatory read is what stops it.

## The skill that builds skills

`skill-creator-skill` is on version 3.8. Each version exists because real use found a real gap: vague trigger rules, no token-budget check, structure that nobody validated until a skill broke in the field. Adding a skill goes through the same validation as everything else, so new capabilities don't degrade the old ones.

## Built with

Markdown skill files, Claude, and Claude Code.

---

Jordan Waxman · [LinkedIn](https://linkedin.com/in/waxmanjordan) · [GitHub profile](https://github.com/MrMinor-dev)
