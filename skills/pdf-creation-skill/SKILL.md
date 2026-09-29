---
name: pdf-creation-skill
description: "Create PDF documents. Triggers: 'create PDF', 'PDF report', 'PDF proposal'."
---

# PDF Creation

**Version:** 1.0 | **Layer:** AOS

Create professional PDF documents using reportlab with custom flowables, proper page management, and visual QA.

## Prerequisites

Read `/mnt/skills/public/pdf/SKILL.md` for library reference. This skill adds the agentic process layer for creating polished multi-page documents.

## Workflow

### 1. GATHER
- Read source content and confirm document type (proposal, report, guide)
- Identify diagrams to recreate, visual elements needed
- Confirm: audience, tone, what to include/exclude

### 2. DESIGN SYSTEM
Define before writing code:
```python
C_PRIMARY = HexColor("#1A6B7C")
C_ACCENT = HexColor("#D4793A")
# ... full palette matching any companion deliverables (deck, etc.)
```
- **Typography:** Helvetica family (built-in, no font install needed). Size scale: titles 18-26pt, body 9.5pt, captions 8.5pt.
- **Page layout:** letter size, 0.85" margins, justified body text.

### 3. BUILD

**Script structure:**
1. Custom Flowable classes (accent bars, callout boxes, diagrams)
2. Style definitions (ParagraphStyles with `keepWithNext=1` on ALL headers)
3. NumberedCanvas subclass for page footers
4. Content assembly with story list
5. `doc.build(story, canvasmaker=NumberedCanvas)`

**Critical patterns:**

**Headers — prevent orphaning:**
```python
styles.add(ParagraphStyle('SubHead', ..., keepWithNext=1))
```
Every header style (SectionLabel, SectionHead, SubHead, SubHead2) MUST have `keepWithNext=1`.

**Custom flowables for visual elements:**
- `AccentBar` — colored horizontal rule. Set `self.keepWithNext = True`.
- `QuoteBox` / `WideQuoteBox` — dark background callout with white italic text.
- Diagram flowables — override `draw()` with canvas operations.

**Spaced letter headers (section labels):**
```python
nbsp_text = text.replace(' ', '&nbsp;')  # prevents space collapse
```

**Page breaks — use sparingly:**
- Forced PageBreak only for major section starts (Section 1, Section 2).
- Let reportlab flow content naturally between subsections.
- Use `spacer(16)` between major sections instead of PageBreak when half-page gaps result.

**Diagrams with canvas drawing:**
- Use `self.canv` for direct drawing (roundRect, circle, line, bezier paths).
- For circular loop diagrams: place nodes on a circle using trig, draw arc arrows along a slightly larger radius.
- Arrow arrowheads: compute tangent direction at arc endpoint, draw filled triangle.
- Always set `self.height` accurately — reportlab uses it for page fitting.

### 4. QA
```bash
rm -f pdf-pg-*.jpg && pdftoppm -jpeg -r 150 output.pdf pdf-pg
ls -1 "$PWD"/pdf-pg-*.jpg
```
View every page. Check:
- Orphaned headers at page bottoms (header with no content following)
- Text overlapping callout boxes or diagrams
- Diagram elements overlapping each other
- Caption text covered by diagram elements
- Excessive blank space (half-page or more)
- Closing content orphaned on final page

### 5. FIX CYCLE
1. Identify page break issues → adjust spacers, add/remove PageBreak, use KeepTogether
2. Diagram overlaps → adjust radius, node sizes, flowable height
3. **Rebuild → reconvert images → re-inspect** every time
4. Target: no orphaned headers, no overlapping elements, minimal wasted space

### 6. DELIVER
Copy to `/mnt/user-data/outputs/`, present via `present_files`. Jordan downloads manually (binary 0kb bug).

## Known Issues

| Issue | Fix |
|---|---|
| Spaced headers collapse ("TOOLINGPRINCIPLES") | Use `&nbsp;` in Paragraph text |
| Headers orphaned at page bottom | `keepWithNext=1` on ALL header styles |
| KeepTogether pushes entire block to next page | Use sparingly; prefer `keepWithNext` on styles |
| Custom flowable too large for frame | Reduce `self.height`, check against frame height |
| Unicode subscripts render as black boxes | Use `<sub>` / `<super>` tags in Paragraphs |
| Closing paragraphs orphaned on last page | Reduce spacers above closing section |

## Dependencies
- pip: `reportlab`, `svglib` (optional, for SVG import)
- System: pdftoppm (poppler, for visual QA)

## Authority
Tier 3 (Autonomous) — creation/QA. Tier 2 (Inform) — delivery.
