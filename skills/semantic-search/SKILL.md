---
name: semantic-search
description: "Search MRMINOR knowledge base for relevant context. Use BEFORE Plan Mode on every task. Also use when: encountering unfamiliar conventions, touching multiple workstreams, unsure which doc has needed info, or verifying existing patterns before building."
---

# Semantic Search

Query the MRMINOR semantic index to gather context before building. This is a read-only operation — zero risk, high value.

## When to Search

**Always (mandatory):**
- Before Plan Mode on every task — search 2-3 queries related to the task domain
- Before creating or modifying any workflow, schema, or doc

**Situational:**
- Encountering a convention or pattern you haven't seen before
- Task touches multiple workstreams (e.g., AOS + a former e-commerce project)
- Need to verify what already exists before building something new
- Error messages or test failures reference unfamiliar components

## How to Search

```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "search terms here", "limit": 5}'
```

### Query Formulation
- **3-8 words.** Nouns and specific terms, not sentences.
- **Multiple narrow searches > one broad search.** Run 2-3 targeted queries.
- **Use domain concepts:** `workflow audit standards`, `supabase schema products`, `email router architecture`
- **Don't use file paths** — the index works on content, not paths.

### Examples by Task Type

| Task | Good Queries |
|---|---|
| Building a workflow | `{domain} workflow patterns`, `n8n error handling conventions` |
| Schema change | `supabase schema {table}`, `RLS policy patterns` |
| Doc update | `{doc topic} current status`, `{workstream} architecture` |
| New feature | `{feature area} existing implementation`, `{feature area} conventions` |

## Interpreting Results

| Similarity | Meaning | Action |
|---|---|---|
| >0.6 | Strong match | Use chunk content directly |
| 0.4-0.6 | Good match | Likely relevant, verify with full doc if needed |
| 0.3-0.4 | Weak match | Try different terms |
| <0.3 or empty | No match | Broaden query or fall back to known file paths |

## Using Results

**Use chunk content directly.** The chunk contains the information you need.

Only load the full file when:
- Chunk references "see above" or "continued below"
- You need surrounding context not captured in the chunk
- Multiple chunks from same file need reconciliation

**Anti-patterns:**
- Searching AFTER building (too late — you already missed context)
- Loading full file after search "just to be sure" (wastes context window)
- One giant broad query instead of 2-3 targeted ones
- Skipping search because the task "seems simple" (simple tasks still have conventions)

## Integration with Plan Mode

1. Receive task prompt
2. **Search** — run 2-3 semantic queries based on task domain
3. Review search results for relevant conventions, patterns, existing implementations
4. Enter Plan Mode with search context informing your plan
5. Present plan to Jordan for approval
6. Execute

Search results should visibly influence your plan. If search reveals an existing pattern, your plan should reference it. If search reveals a convention, your plan should follow it.
