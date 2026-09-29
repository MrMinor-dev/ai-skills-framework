# CC Skill Building Patterns

Patterns for creating Claude Code extensibility files: skills, agents, and rules. Learned from that session buildout (3 skills, 2 agents, 1 rule deployed).

---

## Three CC Extensibility Types

| Type | Location | Purpose | Loaded When |
|---|---|---|---|
| **Skill** | `~/.claude/skills/{name}/SKILL.md` | Workflow procedures — step-by-step how-to for a task type | CC works in matching directory or task matches description |
| **Agent** | `~/.claude/agents/{name}.md` | Role-scoped subagents — fresh context with restricted tools and identity | Orchestrator spawns via `Task` tool |
| **Rule** | `~/.claude/rules/{name}.md` | Convention enforcement — always-on constraints for matching paths | Automatically when working in matching path globs |

**Key difference from COO skills:** CC extensibility files are written directly to `~/.claude/` via Desktop Commander. No packaging, no `.skill` zip, no Claude Desktop install step. They're live immediately.

---

## CC Skill Structure

```
~/.claude/skills/{skill-name}/
├── SKILL.md                    ← Required: workflow + context
└── references/                 ← Optional: loaded via @path imports
    ├── {detail-topic-1}.md
    └── {detail-topic-2}.md
```

### Frontmatter (required)
```yaml
---
name: {skill-name}
description: "{What it does}. Activates when {trigger condition}."
---
```

**Description rules:**
- Must describe when the skill activates (CC auto-discovers skills based on task context)
- No "Use when:" prefix like COO skills — CC skills use natural language activation
- Be specific: "Build or modify n8n workflows from a COO prompt spec via REST API" not "Helps with workflows"

### Body Structure
```markdown
# {Skill Name}

{1-2 sentence purpose}

## Context
{What CC needs to know — who provides input, what the input looks like, where outputs go}

## {Main Workflow / Steps}
{Numbered steps with clear actions}

## Rules
{Hard constraints — things that must always be true}
```

### Progressive Disclosure via @imports
CC skills use `@path` references to load detail on demand:
```markdown
**API reference:** @AOS\Domains\Infra\N8N-API-REFERENCE.md
**Anti-patterns:** @references/n8n-anti-patterns.md
```

- `@references/` = relative to skill folder (skill-specific detail)
- `@...` = absolute Drive path (shared SSOT docs)
- Keep SKILL.md lean — move patterns, checklists, and examples to references/

---

## CC Agent Structure

Agents are single `.md` files at `~/.claude/agents/{name}.md`.

### Frontmatter (required)
```yaml
---
name: {agent-name}
description: "{What it does}."
archetype: {implementer|reviewer|researcher|planner}
model: {sonnet|haiku|opus}
tools:
  - {tool1}
  - {tool2}
skills:
  - {skill-name}
rules:
  - {rule-name}
---
```

### Frontmatter Field Guide

| Field | Purpose | Options |
|---|---|---|
| `archetype` | Shapes CC's behavior expectations | `implementer` (builds things), `reviewer` (evaluates quality), `researcher` (gathers info), `planner` (designs approaches) |
| `model` | Token economics | `haiku` (cheap, simple tasks), `sonnet` (default, most tasks), `opus` (deep reasoning only) |
| `tools` | Restricts available tools | `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `Task` (spawns subagents) |
| `skills` | Auto-loads these skills | Reference by `name` field from skill frontmatter |
| `rules` | Auto-loads these rules | Reference by filename (without `.md`) |

### Body Structure
```markdown
# {Agent Name}

{1-2 sentence identity statement}

## Identity
{What this agent IS and IS NOT. Clear boundaries prevent scope creep.}

## What You Receive
{Input contract — what the orchestrator passes to this agent}

## How You Work
{Numbered steps — the agent's execution loop}

## Rules
{Hard constraints specific to this agent's role}

## Output Format
{Exact structure the agent returns to the orchestrator}
```

### Agent Design Patterns

**Builder-Auditor pair:** Separate build and review into two agents with different contexts. The auditor never sees the builder's reasoning — fresh eyes catch what builders rationalize away. Both share the same rule file for consistency.

```
Orchestrator → Builder Agent (implements spec)
                    ↓ (workflow ID)
Orchestrator → Auditor Agent (scores independently)
                    ↓ (JSON score)
Orchestrator → decides: PASS / FIX / ESCALATE
```

**Tool restriction principle:** Give agents minimum necessary tools.
- Builders: Read + Write + Edit + Bash + Grep + Glob (full access)
- Auditors: Read + Grep + Glob + Bash (no Write/Edit — can't accidentally "fix" things)
- Researchers: Read + Bash + Grep (read-only + API calls)

**Model selection:**
- `haiku` for simple, repetitive tasks (data extraction, formatting)
- `sonnet` for most work (building, auditing, testing) — default
- `opus` only when deep architectural reasoning is needed — expensive, use sparingly

---

## CC Rule Structure

Rules are single `.md` files at `~/.claude/rules/{name}.md`.

### Frontmatter (required)
```yaml
---
paths:
  - "{glob-pattern-1}"
  - "{glob-pattern-2}"
---
```

### Path Scoping
Rules only activate when CC is working in files matching the glob patterns:
- `"**/AOS/Workflows/**"` → any file in any Workflows directory
- `"**/AGENCY/**"` → Agency files only
- `"**/*.py"` → all Python files

### Body Structure
```markdown
# {Convention Name}

Rules for {scope}. These are conventions, not procedures — skills and prompt files provide step-by-step instructions.

## {Category 1}
- {Rule with rationale}

## {Category 2}
- {Rule with rationale}
```

**Rules vs Skills:** Rules are always-on constraints (like guardrails). Skills are step-by-step procedures (like playbooks). If it's "always do X when in this context" → rule. If it's "here's how to accomplish Y" → skill.

---

## Deployment Workflow

### 1. COO drafts the file
Write content in the COO session. Validate against patterns above.

### 2. Write to ~/.claude/ via Desktop Commander
```
Filesystem:write_file → ~/.claude/skills\{name}\SKILL.md
Filesystem:write_file → ~/.claude/agents\{name}.md
Filesystem:write_file → ~/.claude/rules\{name}.md
```

Create directories first if needed:
```
Filesystem:create_directory → ~/.claude/skills\{name}\
Filesystem:create_directory → ~/.claude/skills\{name}\references\
```

### 3. Backup to Drive
Copy to `AOS\CC-Config\` preserving structure. Update `AOS/CC-Config/MANIFEST.md`.

### 4. Update docs
- `session-context.md` — note what was deployed
- `SKILLS-ROADMAP.md` — add/update CC skill entry
- `claude-instructions.md` — if CC skill count or capabilities changed, update reference section

### 5. Verify in next CC session
CC should be able to discover and use the skill/agent/rule. If it doesn't activate, check:
- Frontmatter format (YAML must be valid)
- Description/path matching (does the trigger condition match?)
- File location (must be exactly `~/.claude/skills/`, `agents/`, or `rules/`)

---

## Cross-Reference Patterns

CC skills, agents, and rules form a dependency graph:

```
Rule (always-on conventions)
  ↑ referenced by
Agent (role-scoped subagent)
  ↑ spawned by orchestrator, uses
Skill (workflow procedures)
  ↑ references
Drive docs (SSOTs — API refs, schema docs, domain docs)
```

When creating a new capability:
1. Does it need always-on constraints? → Write a **rule** first
2. Does it need a step-by-step procedure? → Write a **skill** referencing the rule
3. Does it need a dedicated subagent role? → Write an **agent** referencing both

Not every capability needs all three. A rule alone is fine for simple conventions. A skill alone is fine for procedures without role separation.

---

## Quality Checklist (CC Files)

- [ ] Frontmatter valid YAML with all required fields
- [ ] Description specific enough for CC to auto-discover
- [ ] @path references resolve to existing files
- [ ] Agent tools restricted to minimum necessary
- [ ] Rule path globs match intended scope (not too broad)
- [ ] No secrets or credentials in file content
- [ ] Backed up to `AOS/CC-Config/`
- [ ] Cross-references between skill↔agent↔rule are bidirectional
