---
name: session-startup-skill
description: "Lightweight mid-day session reload. Use when: start session, new session, continuing work mid-day."
updated: 2026-05-29
---

# Session Startup

Lightweight reload for mid-day sessions. Get back to work fast.

## Quick Reference

This is a *reload*, not a briefing. Show what's next on the build queue, what happened last session, and let Jordan pick a lane. Don't pre-load workstream docs — route after selection. For first-session-of-day, use `session-start-day-skill` instead.

## Workflow

### 1. GATHER KNOWLEDGE
Read session context from Jordan's computer:
```
Filesystem:read_file → HAIOS\session-context.md
```

**Critical:** Use Desktop Commander / Filesystem tools, NOT Google Drive search or semantic search.

### 1.5. QUICK CHECK (CONTEXT-ECONOMY v1.3 §Session-Start)
Run silently after reading session-context.md. Takes ~30 seconds. Surface findings in the ANNOUNCE block only if actionable.

1. **Storm flag.** Check `Filesystem:read_file → ~/.claude/STORM-DETECTED.flag`. If the file EXISTS: stop — announce the flag prominently at the top of the session, route to `cc-audit-skill` before any other work. Flag must persist until audit is written.
2. **session-context.md size.** Count lines in the file just read. If > 130: note “Session context over hard max ({N} lines) — session-end pruning may not have fired last session.” Surface in ANNOUNCE.

### 1.6. PREMISE SCAN (CONTEXT-CONTINUITY v1.3 §Premise Re-Examination)
Run silently after QUICK CHECK. Targets the bundled-hold failure mode (n=3): prior-session framing carries unchallenged into the new session, leading to operational decisions made under stale or false premises. Most sessions will be no-ops; the cost of running is low and the cost of skipping is high.

**Scope:** NEXT SESSION, FLAGS, WORKSTREAMS lines from session-context.md containing any of: `hold`, `gate`, `blocks`, `blocked by`, `gated by`, `depends on`, `deferred until`, or classification claims (skill class, doc type, ownership track, "new class", "taxonomy").

**For each match, run the premise test:**
1. State the inherited claim: *"Prior session-context says X gates Y"* (or holds/depends/classifies).
2. Derive the mechanism from current state, NOT from the prior framing: *"Why does X gate Y today? What's the actual mechanism?"*
3. Compare:
   - **Match** (mechanism is real and current) → claim stands, no surface.
   - **Drift** (mechanism is stale, weaker than claimed, or never existed) → flag for ANNOUNCE.

**Surface in ANNOUNCE block** (only if drift found):
```
⚠️ Premise re-examination: [N items]
- [Item]: claim "[inherited framing]" — current mechanism "[what's actually true]"
```

**Do NOT auto-act on drift.** SURFACE to Jordan. Jordan adjudicates whether the inherited framing is wrong, partially-wrong, or actually correct. Acting on drift without confirmation re-creates the same failure mode in reverse.

**Skip silently** when zero claims match scope keywords — most short session-contexts will produce no findings.

### 1.7. ACTIVE DISCOVERY WEEK CHECK
Run silently after PREMISE SCAN. If session-context.md FLAGS contains `discovery week LIVE`:
1. Identify today's DAY-N file path from the active engagement entry in FLAGS (e.g., `AGENCY/Prospects/[CLIENT]/discovery-week/DAY-N-YYYY-MM-DD.md`)
2. Read: `Filesystem:read_file → [derived path]`
3. Extract the **Day N+1 Observation Targets** section
4. Surface in ANNOUNCE block as: `📋 [Client] Day [N] targets: [list]`

Skip silently if no active discovery week in FLAGS.

### 2. ANNOUNCE
```
**Session [N] Starting** [recommend: Sonnet | Opus]

[If premise drift flagged in step 1.6:]
⚠️ Premise re-examination: [N items]
- [Item]: claim "[inherited framing]" — current mechanism "[what's actually true]"

Last session ([N-1]): [one-line from RECENT SESSIONS]
Build queue next: [top unblocked item from NEXT SESSION priorities]

Continue with [next build item], or something else?
1) Continue build queue
2) Agency topic
3) AOS-HAIOS
4) Other
```

**Premise re-examination block:** Include only if step 1.6 PREMISE SCAN flagged drift. If zero items, omit the block entirely — don't add noise to a clean announce.

**Model recommendation:** Check [O]/[S]/[H] tags on NEXT SESSION items.
- Majority [S]/[H] → "Start in Sonnet — ops queue today."
- Mix of [S] and [O] → "Start Sonnet for N ops items, switch to Opus for [item]."
- Majority [O] → "Start in Opus — strategy work."
- No tags → default Sonnet.

**Pending skill installs:** If session-context.md has entries under NEXT SESSION → "Skill installs pending", surface the count in the announce block so Jordan knows Claude Desktop is running stale versions of those skills. Example: "⚠️ 2 skill installs pending — see session-context."

### 3. ROUTE RESPONSE
Based on Jordan's selection, gather context efficiently:

| Selection | Context Gathering |
|---|---|
| **Continue build queue** | Read master index at `HAIOS/Architecture/BUILD-ROADMAP.md` to identify which domain is relevant. Infrastructure and GCT queues are inline in BUILD-ROADMAP.md (consolidated). Then load the appropriate domain roadmap for unblocked items: `AGENCY/AGENCY-BUILD-ROADMAP.md`, `AOS/Strategy/BUSINESS-OPS-BUILD-ROADMAP.md`, or `AOS/Strategy/APP-FACTORY-BUILD-ROADMAP.md`. If routing to a Build or Dev task: also load `HAIOS\Knowledge\COO-REFERENCE-MANUAL.md` via `Filesystem:read_text_file` (fallback when AOS semantic search is unavailable). |
| **Agency topic** | Read Agency section from session-context.md. Use semantic search for specific client/engagement context if needed. |
| **AOS-HAIOS** | Read HAIOS + AOS sections from session-context.md. Check `AOS/Domains/SERVICE-CATALOG.md` for current state. Also load `HAIOS\Knowledge\COO-REFERENCE-MANUAL.md` via `Filesystem:read_text_file` (fallback when AOS semantic search is unavailable). |
| **Other** | Ask Jordan what they need. Load context on demand — PB, Job Apps, etc. |

**After routing:** Only load docs relevant to the selected area. Don't pre-load everything.

## Handoff Contract

When complete, state:
- Session number announced
- Focus area selected: [area]
- Ready for work

## Dependencies

- Required: Desktop Commander / Filesystem tools (`read_file`)
- Required: `session-context.md` on Drive
- Optional: Semantic search workflow (for deeper context after focus selected)
- Optional: Domain roadmaps (loaded only if continuing build queue)

## Authority

Tier 3 (Autonomous) — read-only operations

## Error Handling

- **File not found:** Ask Jordan to verify path
- **Desktop Commander unavailable:** Cannot proceed; inform Jordan
- **Malformed file:** Report issue, ask Jordan to check

## Cross-References

- Related: `session-start-day-skill` (first-session-of-day full briefing)
- Related: `session-end-skill` (writes session-context.md this skill reads)
- Master index: `HAIOS/Architecture/BUILD-ROADMAP.md`
- Domain roadmaps: `AGENCY/`, `AOS/Strategy/`, `HAIOS/Architecture/`

## Notes


- Multiple areas in one session is fine — re-route when Jordan switches topics
- **Skill storage doctrine:** All skills live canonically at `AOS/Skills/[skill-name]/`. `HAIOS/Skills-Updates/` is deprecated — staging now happens via session-context.md "Skill installs pending" list.
- **Doctrine: new task = new session**. This skill is one half of the implementation — a "start session" trigger always begins fresh. No context carries over from previous sessions; handoff happens via `session-context.md`. Full guidance: `CC-DEVELOPMENT-PLAYBOOK.md` § Context Management.
