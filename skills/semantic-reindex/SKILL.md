---
name: semantic-reindex
description: "Incrementally update the semantic search index after creating or modifying .md files in MRMINOR/. Never does a full rebuild — only processes specified or changed files."
---

# Semantic Reindex

Incrementally update the semantic search index for changed files. This is NOT a full rebuild — it only processes the files you specify (or detects changed via hash comparison).

## Usage

### Incremental with known files (preferred — fastest)
```powershell
cd mrminor-mcp; py semantic_index.py --files "relative/path/to/file1.md" "relative/path/to/file2.md"
```

Paths are relative to `<drive-root>/`. Example:
```powershell
cd mrminor-mcp; py semantic_index.py --files "AOS/Workflows/AOS-Email-Email-Router/README.md" "AOS/Domains/INTELLIGENCE-DOMAIN.md"
```

Only the listed files are re-chunked and re-embedded. Everything else untouched.

### Smart incremental (fallback — slower but catches everything)
```powershell
cd mrminor-mcp; py semantic_index.py
```
Compares file hashes against last indexed state. Only processes files whose content changed since last run. Still incremental — not a full rebuild.

### NEVER do a full rebuild unless Jordan explicitly requests it.

## Verify

After reindexing, confirm a unique term from a changed file appears in search results:
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "{unique term from changed doc}", "limit": 3}'
```

## When to Invoke

- After the build→audit loop completes and docs were updated
- After writing or updating any README in `AOS/Workflows/`
- After updating domain docs, schema docs, or architecture docs
- Batch at task completion — NOT after every small edit mid-task

## Output

Report to orchestrator:
```
Reindexed: {n} files
Verified: "{search term}" → {file} found in results
```
