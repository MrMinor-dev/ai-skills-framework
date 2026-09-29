---
name: wikify-skill
description: "Wikify raw assets into client wiki pages using the 4-step Extract→Route→Generate→Validate flow. Use when: wikify, wikifying, update knowledge base, wikify [client] [asset], run wikifier."
---

# Wikify Skill

**Version:** 1.0 | **Session:** 549

COO-executes wikification in-session ($0 incremental cost). Turns raw assets (transcripts, whiteboards, notes, articles) into compiled wiki pages for a client's `Knowledge/` directory. Phase 1 of the knowledge architecture — proves the 4-step pattern before automating in n8n.

## Quick Reference

4 steps, each independently verifiable: **Extract** (what's in the asset?) → **Route** (which pages?) → **Generate** (write the pages) → **Validate** (coverage + consistency). Partial success is acceptable — write completed pages, report what failed.

**Asset location:** `AGENCY/{Client}/Call-Notes/` or `AGENCY/{Client}/Assets/`  
**Wiki location:** `AGENCY/{Client}/Knowledge/`  
**Authority:** Tier 2 — writes to Drive, inform Jordan after.

---

## Workflow

### 0. INTAKE

Confirm from Jordan (or infer from context):
- **Client shortcode** (PL, Adaptig, agency, etc.)
- **Asset path** (full Drive path, or "latest transcript" → find it)
- **Asset type** (transcript, whiteboard, article, note, email)

Read the asset:
```
Filesystem:read_file → AGENCY\{Client}\{path-to-asset}
```

If path unknown: `Filesystem:list_directory → AGENCY\{Client}\Call-Notes\` to find it.

---

### 1. EXTRACT

Read the asset. Produce a numbered extraction list:

```
For each meaningful unit in the asset:
- #N | TYPE: fact | decision | action_item | signal | relationship
- CONTENT: concise self-contained statement
- SOURCE: brief quote or location marker ("~min 12", "section 3")
- CONFIDENCE: high | medium | low
```

Log total extraction count. Flag if < 5 — asset may be sparse or already processed.

---

### 2. ROUTE

Load client's page registry:
```
Filesystem:read_file → AGENCY\{Client}\Knowledge\PAGE-REGISTRY.md
```

For each extraction, assign to target page(s) from the registry. Identify:
- Pages to **update** (exist, need new content integrated)
- Pages to **create** (new, not in registry)

Load existing content for pages being updated (contradiction detection in step 4):
```
Filesystem:read_multiple_files → [list of pages being updated]
```

Output: routing table (extraction # → page name(s)).

**Standard page types** (reference when creating new pages):

| Page | Updated After |
|---|---|
| `{CLIENT}-BUSINESS-CONTEXT` | Discovery + ongoing signals |
| `{CLIENT}-AI-ROADMAP` | Whiteboard output, strategic shifts |
| `{CLIENT}-DECISION-LOG` | Every call, every decision (append-only) |
| `{CLIENT}-RELATIONSHIP-MAP` | New people, communication prefs |
| `{CLIENT}-STRATEGY-CURRENT` | Monthly or major pivots |
| `{CLIENT}-SKA-AUDIT` | Skills/Knowledge/Abilities assessment |
| `{CLIENT}-INDUSTRY-INTEL` | 3P intel signals, competitive updates |

---

### 3. GENERATE

For each page in the routing table, draft the updated/new content:

**For updates:**
- Integrate new extractions with existing content
- Prefer more recent source if contradictory
- Add `## Revision History` footer entry: `| S{N} | {date} | {1-line summary} |`

**For new pages:**
- Use standard page type structure (frontmatter + ## sections + Revision History)
- Register in PAGE-REGISTRY.md

**Rules:**
- Obsidian-compatible markdown. Use `[[wikilinks]]` for cross-page references.
- Factual and cited. No editorializing.
- Contradiction → flag explicitly + note superseded content in Revision History.

Partial success OK — complete pages one at a time.

---

### 4. VALIDATE

Run these checks before writing:

| Check | Pass Criteria |
|---|---|
| **Coverage** | Every extraction assigned to ≥1 page |
| **Contradictions** | None unresolved (or explicitly flagged with rationale) |
| **Registry sync** | New pages listed in PAGE-REGISTRY.md |
| **Wikilinks** | Cross-references use registered page names |
| **Traceability** | All content traceable to an extraction (no hallucination) |

Minor failures → correct inline. Major failures → report to Jordan before writing.

---

### 5. WRITE

Write each page to Drive:
```
Filesystem:write_file → AGENCY\{Client}\Knowledge\{page}.md
```

If new pages created, update registry:
```
Filesystem:write_file → AGENCY\{Client}\Knowledge\PAGE-REGISTRY.md
```

Flag for semantic index update at session end (don't run inline — batch it).

---

## Handoff Contract

When complete, state:
- **Client + asset:** [who + what]
- **Pages updated:** [list with full paths]
- **Pages created:** [list with full paths]
- **Extractions:** [N extracted → N routed → N written]
- **Validation:** PASS | PASS WITH FLAGS | FAIL
- **Flags:** [contradictions noted, dropped extractions, anything Jordan should review]
- **Semantic index:** flagged for end-of-session batch

---

## Dependencies

- Required: Asset on Drive (or provided inline in session)
- Required: `AGENCY/{Client}/Knowledge/PAGE-REGISTRY.md` exists
- Optional: Existing page content (for contradiction detection)
- Chains to: `semantic-index-update-skill` at session end
- Chains from: Jordan's direct command, post-call session flow

---

## Authority

**Tier 2 (Inform after)** — writes wiki pages to Drive. Reversible (Drive version history). No spend, no external effects. Inform Jordan of pages changed at completion.

---

## Error Handling

- **Asset not found:** Ask Jordan for path. Do not proceed.
- **PAGE-REGISTRY.md missing:** Stop. Confirm Knowledge/ directory exists for this client. May need to create it.
- **Zero extractions:** Warn Jordan. Asset may be empty, corrupt, or already wikified.
- **Contradiction detected:** Flag explicitly. Never silently overwrite. Prefer more recent source.
- **Partial completion:** Write completed pages. Report what failed + why.
- **Unknown error:** State what happened, what was attempted, ask Jordan.

---

## Cross-References

- Architecture: `HAIOS/Architecture/AGENCY-KNOWLEDGE-ARCHITECTURE.md` — 4-step design, Phase 1/2 deployment
- Registry: `AGENCY/{Client}/Knowledge/PAGE-REGISTRY.md` — page names + status
- Chains to: `semantic-index-update-skill`
- Roadmap: Build Roadmap #23
