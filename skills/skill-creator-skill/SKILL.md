---
name: skill-creator-skill
description: "Create/update/package/install MRMINOR skills. READ THIS FIRST whenever 'skill' appears with any verb (creating, updating, packaging, installing, presenting, handing off, sharing, preparing, shipping, etc.) or whenever operating on SKILL.md or.skill files. Do not rely on enumerated trigger phrases — match on function, not vocabulary. (enforcement: trigger surfaced because 'present the 2 skills to install' was routed to `present_files` instead of this skill's packaging workflow.)"
---

# Skill Creator

**Version:** 3.8

Creates MRMINOR-compliant skills for both platforms and handles the full lifecycle through install, backup, and doc updates.

## Two Skill Platforms

| Platform | Installed To | Packaged? | Triggers |
|---|---|---|---|
| **COO** (Claude.ai) | `/mnt/skills/user/` via `.skill` zip | Yes — Jordan installs in Claude Desktop | `description:` "Use when:" in frontmatter |
| **CC** (Claude Code) | `~/.claude/skills/`, `agents/`, `rules/` | No — write directly via Desktop Commander | Auto-discovered by description + path globs |

**Decision:** If the skill governs strategy, orchestration, or doc management → COO. If it governs code execution, CLI tasks, or build procedures → CC. If unclear, ask Jordan.

## Workflow — COO Skills

> ### CORE RULE: Build in the sandbox → present → THEN sync to Drive.
> A COO skill installs ONE way only: Jordan clicks **"save skill"** on a `present_files` card in chat. A machine path or Drive path CANNOT be installed — never hand Jordan a file path as the install artifact. Therefore the pipeline is fixed:
> 1. **Author/edit** SKILL.md + references **in the sandbox** (`/home/claude/{skill}/...`).
> 2. **`zip -r /mnt/user-data/outputs/{skill}.skill {skill}/`** in the sandbox.
> 3. **`present_files`** the `.skill`. This IS the install handoff. It is **mandatory and automatic on every create/update** — Jordan never has to ask for packaging.
> 4. **THEN** write the canonical SKILL.md + references to Drive via `Filesystem:write_file` (Drive = SSOT).
>
> **Why this order:** content originates in the sandbox, so `present_files` can reach it directly and the zip computes its own CRCs over known-good bytes — nothing to corrupt. The sandbox and Jordan's Drive are **separate filesystems** (sandbox = Linux container; Drive = Windows via Desktop Commander), with no shared mount and no Drive access from the sandbox network.
>
> **NEVER author on Drive first, and never move bytes through the chat context.** A base64 round-trip of a Drive-resident file corrupts: hand-copying dense base64 flips a character, the zip fails CRC, and re-typing reproduces the same error (burned an entire session; only fixed by abandoning the bridge and rebuilding in the sandbox from clean text).
>
> **The ban is on bytes-through-context, not on Drive→sandbox transfer as such.** `Filesystem:copy_file_user_to_claude` moves a file out-of-band — no bytes enter the context, nothing is transcribed, so the failure mode is structurally absent. It is the correct way to pull a Drive-canonical file in for an **update** (Step 4). Authoring still starts in the sandbox for new skills; Drive is still written last (Step 7).

### 1. GATHER KNOWLEDGE
- Check `AOS/Skills/SKILLS-ROADMAP.md` for existing skills and backlog
- Review a similar existing skill for structure reference
- Read `references/SKILL-TEMPLATE.md` for canonical structure

### 2. REQUIREMENTS
Confirm with Jordan:
- What should this skill do?
- What triggers it? (specific phrases for claude-instructions.md)
- What layer? Default: **AOS** (`AOS/Skills/[name]/`). Exception: strictly business-coupled skills go to `[Business]/Skills/` (PB, etc.). If unclear, consult `doc-management-skill/references/LAYER-TAXONOMY.md`.
- If the skill produces structured phase outputs (audit phases, pipeline stages, multi-deliverable workflows), enumerate every required deliverable template now and flag gaps before packaging. (Q2-2026 finding: a missing Phase 7 synthesis template persisted many sessions.)

### 3. PLAN
- Sketch outline, estimate size (<500 lines for SKILL.md)
- Identify dependencies (other skills, tools, libraries)
- Map to existing patterns (phased workflow, gather→build→verify, etc.)

### 4. CREATE — IN THE SANDBOX
Build the skill tree under `/home/claude/{skill-name}/` — this is the build + package location. Drive is written later (Step 7), not now.

**Structure:**
```
/home/claude/{skill-name}/
├── SKILL.md           ← Lean workflow
└── references/        ← Optional: details loaded on demand
```

- **New skill:** author `SKILL.md` (and any `references/*.md`) directly into the sandbox tree with `create_file`. For files >~12 KB, write in parts and `cat` them together (`create_file` caps each write near 12 KB and cannot append).
- **Updating an existing skill** (canonical lives on Drive): pull each file in with **`Filesystem:copy_file_user_to_claude`** — one call per file, byte-exact, no transcription, no size cap. Then `cp` from the landing dir into the build tree:
  ```bash
  # copy_file_user_to_claude lands every file FLAT at /mnt/user-data/uploads/{basename}
  mkdir -p /home/claude/{skill-name}/references
  cp /mnt/user-data/uploads/SKILL.md /home/claude/{skill-name}/SKILL.md
  cp /mnt/user-data/uploads/{ref}.md /home/claude/{skill-name}/references/{ref}.md
  ```
  Edit the changed file in the build tree with `str_replace`. Unchanged reference files ride across verbatim so the package is complete. **Do not read the file into context and retype it, and do not edit on Drive first** (see CORE RULE).
  - **Verify:** `sha256sum` the build tree against the Drive originals before packaging. Unchanged files must match; the edited file must differ only where intended.
  - **Landing dir is flat and read-only.** `/mnt/user-data/uploads/` keeps basename only — two files named `SKILL.md` from different skills collide. Do one skill at a time, and always `cp` into the build tree rather than zipping from uploads.
- **Functional code in references** (e.g. a docx template `.js`): after writing it to the sandbox, render-test it (`node {file}`) so a transcription error surfaces as a crash, not a silent bug.

### 5. VALIDATE
- [ ] Frontmatter has `name` and `description` with trigger phrases
- [ ] Token budget reasonable (SKILL.md readable, not bloated; target <300 lines)
- [ ] **Changelog: max 10 rows, one line each, no inline rationale.** The comment `<!-- MAX 10 ROWS... -->` must be present above the table. Rationale belongs in session records, not the skill.
- [ ] Trigger phrases don't overlap with existing skills
- [ ] Authority tier declared
- [ ] Dependencies listed
- [ ] Workflow steps are actionable (not vague)
- [ ] Any functional reference code render-tested in the sandbox
- [ ] Sandbox tree matches intended package layout (`{skill}/SKILL.md` + `{skill}/references/...`)
- [ ] On updates: `sha256sum` — unchanged files match Drive; packaged bytes match the build tree

### 6. PACKAGE & PRESENT (mandatory + automatic)
This step runs on EVERY create/update without being asked. From the sandbox build tree:
```bash
cd /home/claude
rm -f /mnt/user-data/outputs/{skill-name}.skill
zip -r -X /mnt/user-data/outputs/{skill-name}.skill {skill-name}/
unzip -t /mnt/user-data/outputs/{skill-name}.skill   # must report no errors
```
Then `present_files` the `.skill`. That card is Jordan's install button — the only install mechanism. Use a forward-slash zip (Linux `zip` does this natively; never `Compress-Archive`, which writes backslash paths that violate the zip spec).

### 7. SYNC TO DRIVE (SSOT)
After presenting, write the canonical files to Drive so the sandbox build and Drive stay identical:
```
Filesystem:write_file → AOS\Skills\{skill-name}\SKILL.md
Filesystem:write_file → AOS\Skills\{skill-name}\references\*.md   (only changed files)
```
`write_file` handles large text via chunked append — no size cap concern for `.md`. Queue the write paths for the session-end semantic-index update.

### 8. POST-INSTALL CHECKLIST (Mandatory)
After Jordan confirms install, complete ALL of these:
- [ ] **SKILLS-ROADMAP.md** — Add to Completed table (or update version). Bump doc version. Add changelog entry. This IS the place skill version numbers live.
- [ ] **claude-instructions.md — CONDITIONAL, most updates skip this file entirely.** The Skills table there is `| Skill | Triggers |` ONLY — no version numbers, no changelog, nothing else. Check: did the set of trigger phrases that should fire this skill actually change (new skill added, a trigger phrase renamed/added/removed, or the description's user-facing wording shifted)? **If yes** → update that skill's row (or add a new row) with the current trigger phrases, and bump the file's own top `VERSION:` line (that line tracks edits to claude-instructions.md itself, not the skill's version). **If no** — content-only changes, internal workflow rewrites, new validation gates, bug fixes to the skill's logic — the trigger phrases a user would type are unchanged, so claude-instructions.md is not touched at all.
- [ ] **Prompt Jordan (only if claude-instructions.md bullet above actually fired):** "Please copy the updated claude-instructions.md to userPreferences in Claude Desktop Settings → Profile"

> **A Drive write is not an install (n=2).** Syncing Step 7 changes nothing about what Claude Desktop loads; only the `present_files` card does. Record skill versions as `Drive (Desktop: X)` and stage any un-installed version under session-context "Skill installs pending" until Jordan confirms.

## Workflow — CC Skills (Skills, Agents, Rules)

**Full patterns reference:** `references/CC-SKILL-PATTERNS.md`

### 1. DETERMINE TYPE
| Need | Type | Location |
|---|---|---|
| Step-by-step procedure for a task type | **Skill** | `~/.claude/skills/{name}/SKILL.md` |
| Role-scoped subagent with restricted tools | **Agent** | `~/.claude/agents/{name}.md` |
| Always-on constraints for a path scope | **Rule** | `~/.claude/rules/{name}.md` |

Not every capability needs all three. A rule alone works for conventions. A skill alone works for procedures.

### 2. DRAFT
Write the file content in COO session. Follow patterns in `references/CC-SKILL-PATTERNS.md`:
- Skills: frontmatter (name, description) + Context + Steps + Rules
- Agents: frontmatter (name, description, archetype, model, tools, skills, rules) + Identity + Input/Output contracts
- Rules: frontmatter (paths: glob patterns) + categorized conventions

### 3. DEPLOY
Write directly to `~/.claude/` via Desktop Commander:
```
Filesystem:write_file → ~/.claude/skills\{name}\SKILL.md
Filesystem:write_file → ~/.claude/agents\{name}.md
Filesystem:write_file → ~/.claude/rules\{name}.md
```
No packaging step. Files are live immediately for next CC session.

### 4. BACKUP
Copy to Drive backup: `AOS\CC-Config\` preserving directory structure. Update `AOS/CC-Config/MANIFEST.md`.

### 5. VALIDATE
- [ ] Frontmatter valid YAML with all required fields
- [ ] Description specific enough for CC auto-discovery
- [ ] `@path` references resolve to existing files
- [ ] Agent tools restricted to minimum necessary
- [ ] Rule path globs match intended scope
- [ ] No secrets in file content
- [ ] Backed up to `AOS/CC-Config/`
- [ ] Cross-references between skill↔agent↔rule are bidirectional

### 6. DOC UPDATES
- [ ] `session-context.md` — note what was deployed
- [ ] `SKILLS-ROADMAP.md` — add/update CC entry
- [ ] `claude-instructions.md` — update CC skill count if changed

## Packaging Gotchas (COO)
- **`present_files` only serves `/mnt/user-data/outputs`** (sandbox). It cannot serve Drive or machine paths.
- **`create_file` caps near 12 KB per write and cannot append.** Write in parts → `cat` together for new files. Does NOT apply when pulling Drive files via `copy_file_user_to_claude`.
- **Never move file bytes through the chat context** — not base64, not retyped text. It corrupts.
- **PowerShell via Desktop Commander:** use `-File script.ps1`, not inline `-Command`.
- **YAML-frontmatter colon-space gotcha:** changelog lines with `: ` inside a frontmatter value break YAML parsing. Keep changelogs in the SKILL.md body, not frontmatter values.

## Handoff Contract
When complete, state:
- Skill name, type (COO/CC), and location
- Layer placement (HAIOS/AOS) for COO; file type (skill/agent/rule) for CC
- Status: 📦 Packaged + Presented (COO, awaiting install) | ✅ Deployed (CC) | ✅ Installed (COO)
- Which docs were updated

## Dependencies
- Filesystem tools (write_file, read_file, create_directory)
- `Filesystem:copy_file_user_to_claude` (Drive → sandbox transfer on the update path)
- `create_file` + `zip` + `unzip` (sandbox packaging)
- present_files (COO delivery — the install mechanism)
- Desktop Commander (CC deployment to `~/.claude/`)

## Authority
Skill creation = Tier 1 (approval required per Operating Agreement)

---

## Changelog
<!-- MAX 10 ROWS. One line per entry. No inline rationale. Delete oldest row when adding new. -->

| Version | Session | Change |
|---|---|---|
| 3.8 |  | Changelog size rule added to VALIDATE checklist; SKILL-TEMPLATE.md updated to minimal format. |
| 3.7 | current | claude-instructions.md bullet made conditional — touched only when trigger phrases change. |
| 3.6 |  | Step 4 update path rewritten to `copy_file_user_to_claude` (byte-exact, no context transit). |
| 3.5 |  | Removed 1 stale a former e-commerce project ref. (Never installed — caught.) |
| 3.4 |  | YAML-frontmatter colon-space gotcha added to Packaging Gotchas. |
| 3.3 |  | COO packaging pipeline rewritten to build-in-sandbox; `present_files` is the only install mechanism. |
| 3.2 |  | Deliverable Template Check added to Step 2 for skills with structured phase outputs. |
| 3.1 |  | Step 2 layer guidance explicit (AOS default, business-layer exception). |
| 3.0 |  | CC skill/agent/rule patterns added; `CC-SKILL-PATTERNS.md` reference file. |
