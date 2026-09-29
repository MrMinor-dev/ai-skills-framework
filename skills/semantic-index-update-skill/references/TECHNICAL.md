# Technical Reference

## Script Location

`mrminor-mcp\semantic_index.py`

## Index Architecture

| Component | Value |
|---|---|
| Storage | Supabase `doc_embeddings` table |
| Embeddings | HuggingFace all-MiniLM-L6-v2 (384 dim) |
| Chunks | 500 chars, 50 char overlap |
| Source | `<drive-root>/` |

## Exclusions

Automatically excluded from indexing:
- `Archive/`
- `.git/`
- `__pycache__/`

## Modes

### Incremental (preferred)
```powershell
cd mrminor-mcp; python semantic_index.py --files "file1.md" "file2.md"
```
Indexes only named files. Use when you know what changed.

### Smart Reindex (fallback)
```powershell
cd mrminor-mcp; python semantic_index.py
```
Compares file hashes, only re-indexes changed files. Use when unsure what changed.

### Full Reindex (emergency only)
```powershell
cd mrminor-mcp; python semantic_index.py --full
```
**⚠️ TRUNCATES table first.** Requires Jordan approval. Use only when index is corrupted.

## Mode Selection

```
Know specific files changed?
  YES → --files with filenames
  NO  → smart (no args)

Index corrupted?
  YES → Ask Jordan → --full
  NO  → Never use --full
```

## PowerShell Syntax

**Always use semicolon (`;`) to chain commands.**

```powershell
# ❌ WRONG (bash syntax)
cd mrminor-mcp && python semantic_index.py

# ✅ CORRECT (PowerShell)
cd mrminor-mcp; python semantic_index.py
```
