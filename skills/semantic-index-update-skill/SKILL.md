---
name: semantic-index-update-skill
description: "Update semantic search index. Triggers: files changed this session, after doc edits, stale search results."
---

# Semantic Index Update

Incrementally update semantic search index for changed files.

## Workflow

### 1. CHECK INPUT
Look for file list from context:
- If orchestrator provides file list: Use it
- If invoked directly: Ask what files changed

### 2. GENERATE PROMPT
Output for Jordan to run in Claude Code:

**With known files:**
```powershell
cd mrminor-mcp; python semantic_index.py --files "[file1.md]" "[file2.md]"
```

**Unknown files (fallback):**
```powershell
cd mrminor-mcp; python semantic_index.py
```
Smart reindex compares hashes, only processes changed files.

### 3. AWAIT CONFIRMATION
Wait for Jordan to confirm:
- Chunk count from output
- No errors

### 4. VERIFY
Search for a term unique to a file changed THIS session, via the canonical webhook + curl:
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "[unique phrase from a doc written this session]", "limit": 5}'
```
Confirm the results include the expected file. **Use content written THIS session, not an older term** — an older term returns hits from the previous index state and proves nothing. A reindex claim is not a reindex.

**Never use `n8n:execute_workflow` for AOS services** — webhook + curl only (`claude-instructions` §TOOLS; `semantic-search-skill` §2). Workflow-specific IDs go inactive when their workflow is deactivated; the canonical URL does not. (Stale `{{WORKFLOW_ID}}` execute_workflow call removed.)

## Handoff Contract

When complete, state:
- Files indexed: [list]
- Chunks: [count]
- Verification: [search term] → [file found] ✅

## Dependencies

- Required: Claude Code (execution), `mrminor-mcp\semantic_index.py`
- Related: Often invoked after `session-end-skill`, `doc-management-skill`, `doc-audit-skill`

## Authority

Tier 3 (Autonomous) — read-only index operation

## Error Handling

| Issue | Action |
|---|---|
| Permission denied | File locked — close and retry |
| Search returns stale | Wait 30s, retry different term |
| Script error | Check `references/TROUBLESHOOTING.md` |

## References

- `references/TECHNICAL.md` — Script modes, exclusions, PowerShell syntax
- `references/TROUBLESHOOTING.md` — Common issues, escalation
