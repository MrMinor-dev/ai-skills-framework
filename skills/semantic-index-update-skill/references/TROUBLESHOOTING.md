# Troubleshooting & Verification

## Verification Process

After index runs, always verify before reporting success.

### Step 1: Check Output
Look for in Claude Code output:
- "Total: X chunks from Y files"
- No Python errors
- Completion message

### Step 2: Verify via Search
Use semantic search workflow:
```
Workflow ID: {{WORKFLOW_ID}}
Input: {"query": "unique term from recently changed file", "limit": 5}
```

Expected: Results include the changed file.

### Step 3: Report with Evidence
```
✅ Semantic index updated:
- Files processed: 3
- Chunks indexed: 47
- Verification: Search for "swarm architecture" returned AGENT-ORCHESTRATION.md ✅
```

## Common Issues

| Issue | Solution |
|---|---|
| "Permission denied" on Google Drive | File may be locked by another app - close file, retry |
| Stale results after index | Try different query terms, wait 30s for propagation |
| Script hangs | Check if background Python process running |
| Module not found | Run from correct directory (`mrminor-mcp`) |
| Supabase connection error | Check internet connection, verify credentials |

## When to Escalate

- Repeated permission errors → Check Google Drive sync status
- Index completes but search never finds file → May need full reindex (get Jordan approval)
- Script crashes consistently → Check `semantic_index.py` for bugs
