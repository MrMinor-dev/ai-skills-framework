# 4-Layer Taxonomy

Layer placement rules for MRMINOR documents and skills. Four layers: HAIOS (L0), AOS (L1), Business (L2), Archive (L3).

---

## The Four Layers

```
L0: HAIOS (Human-AI Operating System)
    └── Meta layer: collaboration, sessions, knowledge management
    
L1: AOS (AI Operating System)  
    └── Platform layer: database, workflows, compliance, shared capabilities
    
L2: Business Instances (AGENCY / PB / DENTAL)
    └── Revenue paths, career assets, and parked optionality - each with its own strategy, content, and operations
    
L3: Archive
    └── Historical: superseded docs, old versions, reference material
```

---

## Layer Decision Tree

```
Is this about how human and AI work together?
    YES → HAIOS
    NO ↓

Is this shared infrastructure used by multiple businesses?
    YES → AOS
    NO ↓

Is this specific to one business instance (AGENCY, PB, or DENTAL)?
    YES → That instance folder
    NO ↓

Is this historical/superseded?
    YES → Archive/
    NO → Ask Jordan
```

**Pivot Test:** If MRMINOR pivoted to completely different businesses, would this doc still be needed?
- Yes → HAIOS or AOS
- No → Business layer (AGENCY, PB, DENTAL)

---

## HAIOS — Human-AI Operating System

**Path:** `MRMINOR/HAIOS/`

**Contains:**
- Session management (session-context.md)
- Collaboration norms (CEO-COO-CONTRACT.md)
- Knowledge organization (Best Practices, this skill)
- Meta-skills (session-startup, session-end, doc-management, skill-creator)
- Journey capture and learnings
- Architecture decisions for the HAIOS itself

**Subfolders:**
```
HAIOS/
├── Architecture/        ← System design docs
├── Knowledge/          
│   └── BestPractices/  ← Research-backed standards
├── Skills/             ← Meta/collaboration skills
└── Journey/            ← Session captures, learnings
```

**Key files:**
- `session-context.md` — Current state, next session focus
- `CEO-COO-CONTRACT.md` — Authority tiers, decision rights
- `claude-instructions.md` — COO operating instructions
- `Architecture/SSOT-REGISTRY-MASTER.md` — Authoritative list of all SSOTs across all layers

---

## AOS — AI Operating System

**Path:** `MRMINOR/AOS/`

**Contains:**
- Database schema and operations (Supabase)
- Shared workflow infrastructure
- Compliance (business, content)
- Platform capabilities used across businesses
- SSOTs for operational data

**Subfolders:**
```
AOS/
├── Architecture/       ← Platform design
├── Agents/             ← Agent prompts (wikifier, etc.)
├── Capabilities/       ← Shared platform features (Finance, Compliance)
├── Core/               ← Core cross-cutting standards
├── Domains/            ← Business-domain implementations (Database, Compliance, Infra)
├── Skills/             ← Platform operation skills
├── Templates/          ← Document/workflow/contract templates
└── Workflows/          ← Named workflow folders
```

**Key files:**
- `Domains/Database/DATABASE-DOMAIN.md` — Database domain doc (archived — live DB is authoritative — query live DB for schema)
- `Domains/SERVICE-CATALOG.md` — Platform service catalog

**Note:** The SSOT registry itself (`SSOT-REGISTRY-MASTER.md`) lives in `HAIOS/Architecture/`, not AOS — see HAIOS Key files above.

---

## PB — Personal Brand

**Path:** `MRMINOR/PB/`

**Contains:**
- Personal brand strategy (Jordan's individual presence)
- PB-specific content operations and career assets
- PB workflows

**Subfolders:**
```
PB/
├── Job-Tracker/                        ← Active application tracking
├── Career-Portfolio/
│   └── Story-Mining/                   ← D13-protected evidence, never edit
└── GitHub-Repos/                       ← Public-repo working copies
```

---

## DENTAL

**Path:** `MRMINOR/DENTAL/`

**Status:** Parked optionality — validation-gated on dentist reply.

---

**Note (Jordan ruling Q1):** `Personal/` is outside this taxonomy — it is not a business instance and the Pivot Test does not apply.

---

## Document Type Routing

| Document Type | Location |
|---|---|
| Session context, collaboration | `HAIOS/` |
| Best practices, research | `HAIOS/Knowledge/BestPractices/` |
| Meta-skills (sessions, docs, skills) | `HAIOS/Skills/` |
| SSOT registry | `HAIOS/Architecture/` |
| Database schema | `AOS/Domains/Database/` |
| Platform compliance | `AOS/Capabilities/Compliance/` or `AOS/Core/Compliance/` |
| Business strategy | `[Business]/Knowledge/` |
| Workflow specs | `[Business]/Workflows/[id]/` |
| Business-specific skills | `[Business]/Skills/` |
| MRMINOR-scoped CC rule files (workflow rules, subtree-scoped) | `.claude\rules\` (project-level) |
| Cross-project behavioral CC rules (global, no frontmatter) | `~/.claude/rules/` (user-level) |

---

## Skill Placement

MRMINOR skills fall into **two governance tracks** depending on which AI invokes them. The distinction is architectural, not cosmetic: CC instance config (CLAUDE.md, rules, settings, hooks, skills) is a bundled restore unit on Desktop; COO skills install to a different Desktop path (`/mnt/skills/user/`). originally scoped "all skills → `AOS/Skills/`" without accounting for this split; amended per `HAIOS/Architecture/CANONICAL-GAP-DECISION.md`.

### COO skills (triggered in Claude Desktop sessions, installed at `/mnt/skills/user/[skill-name]/`)

- **Canonical:** `AOS/Skills/[skill-name]/` on Drive (single SSOT).
- **Install flow:** COO edits canonical → `session-end-skill` stages entry under NEXT SESSION → "Skill installs pending" → Jordan installs to Desktop from the canonical path → entry removed on confirmation.
- **Type taxonomy** (meta/collaboration, platform-operations, business-specific) tracked in `claude-instructions.md` Skills table for trigger routing, not by folder.
- **Business-isolated skills** may live at `[Business]/Skills/` if business coupling is strict (instance-specific), but default is `AOS/Skills/`.

### CC-operational skills (triggered by Claude Code during its own sessions)

- **Canonical:** `~/.claude/skills/[skill-name]/` on Jordan's Desktop. CC reads here at session start — local-is-source-of-truth.
- **Drive mirror:** `AOS/CC-Config/skills/[skill-name]/` via `sync-cc-config.ps1` (robocopy `/MIR`, one-way Desktop→Drive).
- **Sync cadence:** session-end (triggered by `session-end-skill` step 8.5, per that session decision). Manual fallback: `run-sync.bat` or `/sync-config` slash command. The mirror is used for audit search and restore — not for direct editing.
- **Install flow:** COO or CC edits local at `~/.claude/skills/` → next session-end sync mirrors to Drive → **no "Skill installs pending" entry needed** (CC reads local, so it's live on save).
- **6 skills in this track** (as of then): `n8n-workflow-build`, `n8n-workflow-audit`, `n8n-diagnose`, `process-feedback`, `semantic-search`, `semantic-reindex`.
- **Rationale for the split:** CC's restore unit = the whole `~/.claude/` tree (CLAUDE.md + rules + settings + hooks + skills + agents). Splitting skills out to `AOS/Skills/` would break the bundle and force a reassembly step on restore. The existing `AOS/CC-Config/` mirror (infrastructure since then) already solves the Drive-backup + semantic-search-coverage needs cleanly.

### Editing rule-of-thumb

| Editing… | Edit at… | Mirror / install |
|---|---|---|
| COO skill (e.g., `doc-management-skill`, `session-end-skill`) | `AOS/Skills/[skill]/` | Jordan installs to Desktop from canonical |
| CC-operational skill (e.g., `n8n-workflow-build`) | `~/.claude/skills/[skill]/` | Session-end sync propagates to `AOS/CC-Config/skills/` |

**Do NOT edit the Drive mirror at `AOS/CC-Config/` directly** — `/MIR` sync will overwrite out-of-band Drive edits at next sync.

**Deprecated:** `HAIOS/Skills-Updates/` was a staging folder for fresh skill updates pending install. Replaced by the session-context.md "Skill installs pending" subsection — which is **COO-skill-only**; CC-operational skills don't use this staging mechanism.

---

## CC Rules Placement

CC rules are markdown files with optional `paths:` / `globs:` frontmatter that load into CC's context to govern behavior. Two placement tiers, selected by scope:

### User-level (`~/.claude/rules/`)

- **For:** Cross-project behavioral rules that apply regardless of which project CC is working in.
- **Frontmatter:** None. Files with `paths:` at user level are silent-dropped per Anthropic bug [#21858](https://github.com/anthropics/claude-code/issues/21858).
- **Load timing:** Every CC session, globally.
- **Example use:** small operational protocols (COO→CC prompt workflow, subagent limitations, OS/env conventions).
- **Examples in use:** `cc-task-prompts.md`, `subagents.md`, `windows-env.md` (~6.8 KB permanent tax).

### Project-level (`.claude\rules\`)

- **For:** MRMINOR-scoped rules — either project-wide or subtree-scoped.
- **Frontmatter options:**
  - **None** — loads every MRMINOR-tree CC session (project-wide unconditional).
  - **`paths:`** (YAML block list, quoted: `paths:\n  - "**/AOS/Workflows/**"`) — loads on Read trigger when a matching file is opened (TRUE conditional).
  - **`globs:`** (comma-separated unquoted: `globs: **/AOS/Workflows/**, **/Workflows/**`) — loads eagerly at session start when cwd is in the project. Behaves as project-wide unconditional regardless of the pattern (empirical finding).
- **Format selection rule:** default to `paths:` for anything subtree-scoped. Only use `globs:` when you explicitly want always-on-in-project. Never use `globs:` for large reference files — they will eager-load.
- **Examples in use:** all 4 workflow rules at project level with `paths:`: `n8n-workflows-core.md`, `n8n-workflows-learnings.md`, `workflow-verification.md`, `n8n-rollback.md` (~41 KB, zero tax on non-workflow MRMINOR sessions).

### Separate-project rules

For projects outside the MRMINOR tree (e.g., `mrminor-mcp\`), create a local `.claude/rules/` at that project root. Independent of MRMINOR rules. Out of scope for this taxonomy.

### Architecture doc

`Archive/PROJECT-LEVEL-RULES-ARCHITECTURE.md` — design, canary evidence, decision log. (Archived flat, not under `HAIOS/Architecture/` as previously cited -- path corrected CC-RULES-CONSOLIDATION.)

---

## Depth Limit

**Maximum depth: 4 levels** (L0-L3)

Per KNOWLEDGE-ORGANIZATION-BEST-PRACTICES.md, deeper nesting harms discoverability. If you need more depth, reconsider organization.

```
✅ HAIOS/Knowledge/BestPractices/FILE.md     (4 levels)
❌ HAIOS/Knowledge/BestPractices/Details/Sub/FILE.md  (6 levels — too deep)
```

---

## Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Putting business docs in AOS | Couples platform to specific business | Move to AGENCY/, PB/, or DENTAL/ |
| Putting shared infra in a business instance | Won't be available to other businesses | Move to AOS/ |
| Creating new top-level folders | Breaks L10 structure | Use existing layers |
| Nesting too deep | Hard to find, poor search | Flatten or split docs |

---

## Changelog

| Version | Date | Session | Changes |
|---|---|---|---|
| 1.1 | 2026-07-15 | (cascade) | L2 spine rewritten to Business Instances (AGENCY/PB/DENTAL). a former e-commerce project section deleted (decommissioned). PB section corrected (was falsely TBD; now reflects live subfolders). DENTAL section added. `Personal/` exclusion note added (Jordan ruling Q1). Common Mistakes + Skill Placement a former e-commerce project/PB refs updated. |
| 1.0 | — | — | Prior baseline (untracked). |
