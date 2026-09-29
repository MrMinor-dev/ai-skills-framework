---
name: semantic-search-skill
description: "Query MRMINOR knowledge base. Triggers: context gathering, workstream switching, before loading docs >100 lines."
---

# Semantic Search

COO's primary context-gathering tool. Search first, load files second.

## Mandatory Triggers (always search)

- **Session startup** — After reading session-context.md, search for selected workstream context
- **Workstream switching** — When Jordan changes focus mid-session, search new workstream before loading docs
- **Any skill's GATHER KNOWLEDGE step** — Search before loading full files
- **Cross-workstream references** — When task touches >1 workstream, search for connections
- **Before loading any doc over ~100 lines** — Search first; chunks may have what you need

## Situational Triggers (search when relevant)

- **Unknown territory** — Don't know which doc contains needed info
- **Verifying current state** — Confirming what exists before building or advising
- **Decision context** — Gathering prior decisions/trade-offs before making new ones

## Search-First Protocol

Default workflow for context gathering:
1. **Search** — 2-3 targeted semantic queries
2. **Read chunks** — Use chunk content directly (don't load full files)
3. **Load file only if** — Chunks reference "see above"/"continued below", or you need reconciliation across chunks from same file

This replaces the old pattern of loading 3+ full files. Chunks are ~750 tokens each vs full files at 500-3000+ tokens.

## Workflow

### 1. FORMULATE QUERY
- Extract key concepts (3-8 words ideal)
- Use nouns and specific terms, not full sentences
- Multiple searches > one broad search

### 2. EXECUTE

```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "[search terms]", "limit": 5}'
```

**Always use the webhook via curl.** Do NOT use n8n:execute_workflow for AOS services.

### 3. INTERPRET
Results ranked by similarity (0.3+ threshold):
- **>0.6:** High confidence — use directly
- **0.4-0.6:** Medium — may need full doc for context
- **<0.4 or empty:** Broaden terms or use known SSOT paths

### 4. ACT — CRITICAL
**Use chunk content directly.** The chunk contains the information you need.

Only load full file when:
- Chunk explicitly references "see above" or "continued below"
- You need surrounding context not in the chunk
- Multiple chunks from same file need reconciliation

**Anti-pattern:** Loading full doc after search "just to be sure" — this wastes tokens and caused compactions.

## Long-Context / RAG Patterns

Retrieved chunks are evidence, not conclusions. Three practices that make retrieved context actually useful vs. noisy (4.7 doctrine, OPUS-4-7 tracker #26):

### Anchors in Documents

Long docs (>200 lines) need navigable structure. When authoring or editing a doc destined for semantic search:
- Use clear `### Section Heading` anchors every ~30-50 lines
- Prefer specific section names over generic ones ("Deployment checklist" > "Process")
- The semantic index chunks on heading boundaries — unclear anchors produce muddy chunks

**Authoring check:** if a section's anchor wouldn't help someone searching "where does X happen in this system," the anchor needs sharpening.

### Prioritize Sections Explicitly

When synthesizing from multiple chunks (one search or several) and some are more trustworthy or recent than others, say so:
- "Primary: X (recent, authoritative). Secondary: Y (dated, may be stale)."
- Priority order for MRMINOR sources: doctrine docs > session-context.md state > RECENT SESSIONS rows > FLAGS > tracker rows > generic chunks

Don't assume the model will infer priority. 4.7's self-verification will silently downweight shaky sources if you let it — but it makes better calls when priority is stated.

### Citations in the Instruction, Not Just Output

When retrieved chunks back a decision or claim, require citations **in the task instruction**, not just the output format:

- **Weak:** "Based on the docs, answer X. Output format: markdown."
- **Strong:** "Answer X. Every factual claim must cite the chunk (filepath + section) that supports it. If no chunk supports a claim, mark it `[no source]`."

Citations-in-instruction triggers evidence-based reasoning *during* the response. Citations-only-in-output triggers retrofit citation *after* the answer is already formed — backwards, and 4.7 is more likely to miss the mismatch because it will confidently cite post-hoc.

## Error Handling

| Error | Cause | Fix |
|---|---|---|
| Empty results | Terms too specific or not indexed | Broaden query, check index freshness |
| Curl timeout | Network/n8n issue | Retry once, then fall back to manual file reads |
| Webhook error | n8n workflow inactive | Check workflow status in n8n |

## Dependencies

- Required: bash_tool, webhook `https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}}`
- Related: `semantic-index-update-skill` (keeps index current)

## Authority

Tier 3 (Autonomous) — read-only, no approval needed
