# Semantic Search Usage
Practical guidance for effective knowledge retrieval.

---

## Query Formulation

**Good queries:**
- `workflow audit standards` (specific concepts)
- `supabase schema products` (domain + entity)
- `session context update pattern` (process + artifact)

**Bad queries:**
- `how do I do the thing with the workflows` (too vague)
- `HAIOS/Skills/skill-creator-skill/SKILL.md` (paths don't work, use concepts)
- `tell me everything about compliance` (too broad)

---

## Similarity Score Interpretation

| Score | Meaning | Action |
|---|---|---|
| >0.7 | Strong match | Use directly |
| 0.5-0.7 | Good match | Likely relevant, verify context |
| 0.3-0.5 | Weak match | May need different terms or full doc read |
| <0.3 | Filtered out | Not returned |

---

## Using Results — CRITICAL

**Default behavior:** Use chunk content directly. Don't load full docs.

**Only load full doc when:**
- Chunk says "see above" or "continued below"
- You need surrounding context not in chunk
- Multiple chunks from same file need reconciliation

**Anti-pattern:** Loading full doc "just to be sure" — wastes tokens, causes compactions.

---

## Common Patterns

**Before creating docs:** Search first to avoid duplication
```
Query: "best practices document creation"
→ Find existing standards before writing new
```

**During skill execution:** Find related context
```
Query: "workflow error handling patterns"
→ Load proven approaches
```

**Unknown territory:** Discover what exists
```
Query: "compliance deadlines tax"
→ Find relevant SSOTs
```

---

## Troubleshooting

| Issue | Fix |
|---|---|
| No results | Broaden terms, try synonyms |
| Too many results | Add specificity, use domain names |
| Stale results | Index may need update |
| Wrong format error | Check curl command structure in SKILL.md |