# GitHub Repo Inventory

**Version:** 2.0
**Last Updated:** 2026-09-29
**Account:** `MrMinor-dev` (github.com/MrMinor-dev). Clones live in `Projects\github\` (not on Drive).

## Live public repos (11)

| Repo | Purpose | Local path |
|---|---|---|
| `MrMinor-dev` | Profile README: headline, system map, links to every repo | `Projects\github\MrMinor-dev` |
| `ai-operating-system` | Hub: architecture, catalog of every skill and workflow, glossary, 13 framework write-ups | `Projects\github\ai-operating-system` |
| `security-governance-framework` | 18 laws and 5 authority tiers, compliance write-up, command hooks, trust and drift SQL | `Projects\github\security-governance-framework` |
| `database-security-framework` | Row-level security, read-only query service, allowlisted write path | `Projects\github\database-security-framework` |
| `n8n-development-framework` | 24 workflow exports and the 65-point audit | `Projects\github\n8n-development-framework` |
| `ai-skills-framework` | 25 versioned skills plus 1 retired | `Projects\github\ai-skills-framework` |
| `rag-knowledge-base` | Semantic search on Postgres and pgvector, indexer, chunking rubric | `Projects\github\rag-knowledge-base` |
| `human-ai-coordination-framework` | State document template, handoff contract, async command schema | `Projects\github\human-ai-coordination-framework` |
| `back-office-automation` | 6 workflows for email, expenses and budget, $0 spending authority | `Projects\github\back-office-automation` |
| `mcp-server-installation-framework` | 4-step protocol for installing MCP servers | `Projects\github\mcp-server-installation-framework` |
| `mrminor-dev.github.io` | Source of the jordanwaxman.com business site and its Cloudflare intake function | `Projects\github\mrminor-dev.github.io` |

Pinned on the profile, in order: ai-operating-system, security-governance-framework, database-security-framework, n8n-development-framework, ai-skills-framework, rag-knowledge-base.

## Private repos (no local clone)

| Repo | State | Note |
|---|---|---|
| `ai-skills-framework-archive` | private, archived | Old version of ai-skills-framework, kept with its history |
| `mcp-server-installation-framework-archive` | private, archived | Old version of mcp-server-installation-framework |
| `semantic-search-transformation` | private, archived | Old version of rag-knowledge-base |
| `compliance-by-design-framework` | private | Merged into `security-governance-framework/COMPLIANCE.md` |
| `mrminor-ops` | private | Operational automation: kill switch and audits (GitHub Actions) |
| `automation-framework`, `production-scaling-flywheel`, `expense-form`, `automated-file-management` | private, archived | Older work |
| `Prompt-Library` | private | Prompt library for AI tools |

The 13 earlier framework repos were deleted from GitHub. Their files are zipped in `Archive\` as `GITHUB-REPO-{name}-DELETED-2026.zip`.

## Rules

- Edit in the local clone, commit, push normally. Never force push.
- Every push: gitleaks on the diff first, and no session counts, internal IDs or dead-venture names in public files.
- Facts (counts of skills, workflows, tables) come from live sources, not from this file.
