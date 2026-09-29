---
name: github-update-skill
description: "Draft GitHub READMEs and CC push prompts. Triggers: 'github update', 'repo readme', 'push to github'."
metadata:
  version: "1.2"
  updated: "2026-06-12 "
---

# GitHub Update Skill

Draft repo READMEs from portfolio deep dives and generate CC prompts for push.

## ⚠️ Source of Truth Rule (MANDATORY)

**Local files ARE the source. GitHub is the mirror.**

| Action | Rule |
|---|---|
| Edit content | Local file FIRST, always |
| Push to GitHub | CC prompt → git push |
| Local ≠ GitHub | Overwrite local with correct content, force push via CC |

**Local file locations:**
- Repo READMEs: `AGENCY\Site\GitHub-Repos\{repo-name}\README.md`
- Site repo (when exists): `AGENCY\Site\jordanwaxman-com\`

**NEVER:** Edit via GitHub web editor, GitHub API, or any direct-to-remote method. If you catch yourself navigating to `github.com/.../edit` — stop, edit the local file instead.

## CC Commit Mechanics — single-file / CI-ops commits

**Scope:** The Source-of-Truth Rule above governs README / portfolio **content** (edit the Drive local file, push via CC). It does NOT cover repo-native **CI / ops files** — GitHub Actions workflows (`.github/workflows/*.yml`), CI configs, Dependabot, etc. — which have no Drive mirror: the repo IS their source. For those, the clean CC-autonomous commit path is the GitHub Contents API via `gh`, no clone required.

**Single-file commit pattern (embed in CC prompts for CI/ops file builds):**
```bash
# Create or update ONE file in a repo without cloning.
# Build a JSON body and pipe it to gh via --input -
#   { "message": "...", "content": "<base64 of file bytes>", "branch": "main",
#     "sha": "<blob sha — include ONLY when UPDATING an existing file>" }
gh api repos/{owner}/{repo}/contents/{path} --method PUT --input -
```
- **content** must be base64-encoded file bytes.
- **sha** is omitted for a NEW file; for an UPDATE, first GET the existing blob sha (`gh api repos/{owner}/{repo}/contents/{path} --jq .sha`) and include it, or the PUT 409s.
- **Auth:** for `MrMinor-dev`-account repos (e.g., `mrminor-ops`), run `gh auth switch --user MrMinor-dev` then **immediately** `gh auth setup-git` — switching the gh user does not update git's credential helper, so a skipped setup-git causes 403s.
- Confirmed CC-autonomous path (`credential-rotation-reminder.yml` committed to `mrminor-ops` via this pattern, SHA 657a493).

**When to use git push instead:** multi-file changes, or any repo with a Drive-side source mirror (READMEs) — those follow the §Source of Truth Rule (edit local → push).

## Workflow

### 1. PICK REPO
Check `references/REPO-INVENTORY.md` for next undone repo.
If Jordan specifies one, use that.

### 2. GATHER SOURCE
Read the deep dive section from Jordan's computer:
```
Filesystem:read_file → PB\Story-Mining\GITHUB-IO-REFRESH.md
```
Find the section matching this repo's deep dive topic.

### 3. DRAFT README
Write README.md following the pattern from `ai-security-compliance-framework`:
- **Hero line:** Bold, specific, production-tested framing
- **The Problem:** Why this matters (3-4 paragraphs)
- **Architecture:** System diagram + component breakdown
- **Key Insight:** The non-obvious lesson learned
- **Results:** Hard numbers from ACCOMPLISHMENT-INVENTORY.md
- **Built With:** Tech stack + session count
- **License / Author:** Standard footer

Save to Jordan's computer (local file = source of truth):
```
Filesystem:write_file → AGENCY\Site\GitHub-Repos\{repo-name}\README.md
```

### 4. GENERATE CC PROMPT
Output a clean prompt Jordan can paste into Claude Code:

**For NEW repos:**
```
Create a new GitHub repo called `{repo-name}` under mrminor-dev.
Initialize with the README from PB\GitHub-Repos\{repo-name}\README.md
Add description: "{one-line description}"
Set topics: ai, {relevant-tags}
```

**For REFRESH repos:**
```
Update the README for github.com/mrminor-dev/{repo-name}
Replace with contents from AGENCY\Site\GitHub-Repos\{repo-name}\README.md
```

### 5. UPDATE INVENTORY
Mark repo done in `references/REPO-INVENTORY.md`.

## Handoff Contract

When complete, state:
- README saved to: `AGENCY/Site/GitHub-Repos/{repo-name}/README.md`
- CC prompt ready (new or refresh)

## Dependencies

- Required: Desktop Commander (read source, write README)
- Required: Jordan executes CC prompt manually
- Source: `PB/Career-Portfolio/Story-Mining/ACCOMPLISHMENT-INVENTORY.md` (numbers)
- Source content: `AGENCY/Site/GitHub-Repos/{repo-name}/README.md` (current live README)

## Authority

Tier 2 — COO drafts autonomously, Jordan approves via CC execution

## Error Handling

- **Source section not found:** Ask Jordan which section maps to this repo
- **Repo already exists on GitHub:** Use REFRESH prompt pattern, not NEW
- **README too thin:** Pull additional evidence from ACCOMPLISHMENT-INVENTORY.md
