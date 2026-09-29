---
title: Semantic Index Scope — What Gets Indexed and Why
doc_type: decision-memo
status: APPROVED
session:-followup
author: COO
created: 2026-04-30
related:
  - mrminor-mcp\semantic_index.py (implementation — EXCLUDE_FOLDERS + EXCLUDE_FILENAME_PREFIXES is authoritative)
  - AOS/Skills/semantic-search-skill/SKILL.md (consumer of this index)
  - AOS/Skills/semantic-index-update-skill/SKILL.md (caller of `semantic_index.py`)
---

# Semantic Index Scope Decision

## METHODOLOGY

This memo codifies which files get embedded into `doc_embeddings` (Supabase) and queried via `semantic-search-skill`, and which do NOT. Decision made-followup after a transcript-included index hit ~390k chunks and dominated search results with conversational filler. Post-decision rebuild: ~9.9k chunks, 97.5% reduction.

---

## 1. The Test

For any file or folder, ask: *would future-COO ever want to semantic-search this and have it surface as a top-5 result for doctrine, decisions, or current-state questions?*

If yes → index. If no → exclude. If "sometimes" → exclude unless the cost of indexing is trivial.

**Semantic search is a doctrine-and-decisions retrieval tool, not a full-corpus archive.** Other tools cover other access patterns: `conversation_search` for chronological transcript recall; `proof-extraction-skill` for structured transcript content; `Filesystem:read_file` + `grep` for full-text search of specific known files; live DB queries for operational state.

---

## 2. Excluded Categories

Codified in `mrminor-mcp\semantic_index.py` `EXCLUDE_FOLDERS` set:

| Folder | Path pattern | Why excluded |
|---|---|---|
| `Transcripts` | `HAIOS/Sessions/Transcripts/` | Conversational reasoning with high noise-to-signal ratio. Distilled content already lives in session-context.md, ANTI-PATTERNS, decision memos, skill files, and proof-extraction structured output. Chronological recall is via `conversation_search`. |
| `Proof-Extraction-Outputs` | `AGENCY/Proof-Extraction-Outputs/` | JSONL audit trails + sample row JSONs are diagnostic data, not search targets. Structured proof/journey content lives in Supabase `haios_*` tables, queried via SQL not semantic search. |
| `Archive` | `Archive/` (and any nested `Archive/`) | Historical artifacts by definition retired. If archived content needs retrieval, grep is the right tool. |
| `_archived` | any `_archived/` dir | Same rationale as `Archive` (used in code repos like `mrminor-mcp\_archived` for old script versions). |
| `.git` | `.git/` | Source control internals. |
| `__pycache__` | `__pycache__/` | Python bytecode cache. |
| `node_modules` | `node_modules/` | NPM dependency tree. |
| `venv` | `venv/` | Python virtual environment internals. |
| `.secrets` | `.secrets/` (defensive) | DOCS_PATH is `<drive-root>/` and `.secrets/` lives there. Hidden-folder rule (`d.startswith('.')`) already excludes it, but explicit listing is documentation. NEVER index credentials. |
| `.claude` | `.claude/` (defensive) | Project-local CC config. Same hidden-folder defense. |

Filename-prefix exclusions (`EXCLUDE_FILENAME_PREFIXES`):

| Prefix | Why excluded |
|---|---|
| `secret-` | Defensive — any future filename mistake that puts a credential in a non-`.secrets/` location still gets blocked. |
| `credential-` | Same. |
| `.env` | Same. |

---

## 3. Included Categories (Strong-Keep)

Anything under `<drive-root>/` ending in `.md` and not matched by the exclusion rules. Notable examples:

- `AOS/Skills/**/*.md` — all skill SKILL.md + references (canonical COO skills)
- `AOS/CC-Config/**/*.md` — CC instance config mirror (skills, agents, rules)
- `AOS/Workflows/**/*.md` — workflow READMEs and CC prompt files
- `AOS/Domains/**/*.md` — service catalog and domain docs
- `HAIOS/*.md` — top-level HAIOS docs (session-context, claude-instructions, CEO-COO contract)
- `HAIOS/Architecture/*.md` — architecture and decision memos
- `HAIOS/Knowledge/**/*.md` — best-practices, reference manuals, anti-patterns
- `HAIOS/Sessions/**/*.md` — EXCEPT `Transcripts/` (insights and session summaries are NOT in transcripts subfolder)
- `AGENCY/*.md` — strategy, plans, ship specs, design memos, active CC prompts
- `AGENCY/[non-Outputs subdirs]/**/*.md` — wikis, contracts, templates

**One borderline case:** `AGENCY/CC-BUILD-LOG-*.md` and `AGENCY/CC-FEEDBACK-*.md` files at AGENCY root currently get indexed. They're verbose post-dispatch artifacts. They get archived after `process-feedback` runs (typically same session), at which point they fall under `Archive/` exclusion. So the indexed-window for any individual log/feedback file is ~one session — acceptable churn, not worth a filename-pattern exclusion.

---

## 4. Implementation

`mrminor-mcp\semantic_index.py` (Supabase-era, post-ChromaDB migration). Key surfaces:
- `EXCLUDE_FOLDERS` set (line ~52) — folder name match against any directory in the walk.
- `EXCLUDE_FILENAME_PREFIXES` tuple (line ~68) — filename prefix match in `get_md_files()`.
- `os.walk` `dirs[:]` mutation (line ~76) — applies folder exclusions in-place to prevent descending into excluded subtrees.
- Hidden-folder rule (`d.startswith('.')`) — applied alongside `EXCLUDE_FOLDERS`. Catches `.git`, `.secrets`, `.claude`, etc., even if not explicitly listed.

**Update protocol:** any change to `EXCLUDE_FOLDERS` or `EXCLUDE_FILENAME_PREFIXES` requires a `--full` rebuild to take effect retroactively. Smart-reindex mode only handles file-level changes (hash check); it does NOT remove already-indexed files that newly match an exclusion rule. Run pattern:
```powershell
cd mrminor-mcp
.\venv\Scripts\python.exe semantic_index.py --full
```

---

## 5. When to Revisit This Scope

Re-evaluate exclusion list if any of:
- Search results regularly miss content the user expected to find → maybe excluded category should be re-included.
- Search results regularly surface low-value content from a category → maybe currently-included category should be excluded.
- New folder type is added to MRMINOR/ that doesn't fit either bucket cleanly → memo update + EXCLUDE_FOLDERS edit.
- Index size grows past ~50k chunks again → likely something got mis-included; check additions since last rebuild.

---

## 6. History

| Date | Session | Change |
|---|---|---|
| 2026-01-21 |  | First exclusion: `Archive/`. Pre-Supabase, on ChromaDB. EXCLUDE_FOLDERS = {Archive,.git, __pycache__, node_modules}. |
| 2026-04-30 | -followup | Added: Transcripts, Proof-Extraction-Outputs, _archived, venv,.secrets,.claude. Added EXCLUDE_FILENAME_PREFIXES tuple. Triggered by 12-hour stuck rebuild that revealed 95%+ of index volume was transcript content. Post-rebuild: 469 files / 9.9k chunks (vs prior ~390k). |
