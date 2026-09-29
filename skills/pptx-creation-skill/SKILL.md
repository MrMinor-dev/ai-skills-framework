---
name: pptx-creation-skill
description: "Create presentation decks. Triggers: 'create deck', 'slides', 'presentation'."
---

# PPTX Creation

**Version:** 1.0 | **Layer:** AOS

Create professional slide decks with a design-system-first approach, visual QA verification, and iterative fix cycles.

## Prerequisites

Always read `/mnt/skills/public/pptx/SKILL.md` and `/mnt/skills/public/pptx/pptxgenjs.md` FIRST — they contain the API reference and common pitfalls. This skill adds the agentic process layer on top.

## Workflow

### 1. GATHER
- Read source content (notes, brief, outline)
- View any reference diagrams/images (copy to Claude's container if on Drive)
- Confirm slide count, audience, design constraints with Jordan

### 2. DESIGN SYSTEM
Define before writing any code:
- **Color palette:** 3-4 colors. Primary, accent, text dark, text light, background. Match the TOPIC.
- **Typography:** Header + body font pairing. Size scale (title 36pt+, body 14-16pt, captions 10-12pt).
- **Layout motifs:** 2-3 recurring patterns (icon rows, 2x2 grids, stat cards). Vary across slides.
- **Dark/light strategy:** Dark bookend slides (title + closing), light content slides.

Store as constants at top of script:
```javascript
const C = { darkBg: "0F1923", primary: "1A6B7C", accent: "D4793A", ... };
const FONT = { head: "Trebuchet MS", body: "Calibri" };
```

### 3. BUILD
Single Node.js script generating the entire deck.

**Critical patterns:**
- **Fresh shadow factory:** `const cardShadow = () => ({ type: "outer", ... })` — PptxGenJS mutates option objects
- **margin: 0** on text boxes when aligning with shapes/icons
- **breakLine: true** between text array items
- **bullet: true** not unicode "•" (double bullets)
- **No # in hex colors** — file corruption
- **No 8-char hex opacity** — use `opacity` property
- **Icons:** react-icons → sharp → base64 PNG for universal compat
- **Speaker notes:** `slide.addNotes()` — depth here, slides stay scannable

### 4. VISUAL QA (Mandatory)
```bash
python /mnt/skills/public/pptx/scripts/office/soffice.py --headless --convert-to pdf output.pptx
rm -f slide-*.jpg && pdftoppm -jpeg -r 150 output.pdf slide
ls -1 "$PWD"/slide-*.jpg
```
View each image. Check: overlaps, clipping, cramped spacing, low contrast, orphaned headers.

### 5. FIX CYCLE
1. List issues (if zero, look harder)
2. Fix via `str_replace` on build script
3. **Rebuild → reconvert → re-inspect** (full pipeline every time)
4. Repeat until clean

### 6. DELIVER
Copy to `/mnt/user-data/outputs/`, present via `present_files`. Binary files can't write to Drive (0kb bug) — Jordan downloads manually.

## Known Issues

| Issue | Fix |
|---|---|
| Shadow corrupts on reuse | Factory function pattern |
| Text misaligned with shapes | `margin: 0` |
| Unicode arrows render inconsistently | Use shapes/lines for diagrams |
| Accent lines under titles | Avoid — AI hallmark |

## Dependencies
- npm: `pptxgenjs`, `react-icons`, `react`, `react-dom`, `sharp`
- pip: `markitdown[pptx]`, `Pillow`
- System: LibreOffice (soffice.py), pdftoppm (poppler)

## Authority
Tier 3 (Autonomous) — creation/QA. Tier 2 (Inform) — delivery.
