---
name: session-start-day-skill
description: "Daily briefing for first session of the day. Use when: start day, morning, good morning, daily briefing, first session of the day."
---

# Session Start Day

Full daily briefing — Jordan opens Claude, absorbs the picture, picks what's first.

## Quick Reference

This is a *briefing*, not a menu. Read everything, present concisely, then ask "What first?" Don't pre-load workstream docs — wait for Jordan's direction. If Google Calendar is not yet integrated, include a manual check reminder.

## Workflow

### 1. GATHER KNOWLEDGE

Read two files from Jordan's computer:
```
Filesystem:read_file → HAIOS\session-context.md
Filesystem:read_file → HAIOS\Architecture\BUILD-ROADMAP.md
```

**Critical:** Use Desktop Commander / Filesystem tools, NOT Google Drive search or semantic search.

### 2. COMPILE BRIEFING

Present in this order:

**A. Today's Date + Session Number**
Increment session number from session-context.md.

**B. Time-Sensitive Flags**
Pull from ⚠️ FLAGS section. Highlight anything dated today, tomorrow, or this week. If nothing time-sensitive, say "No immediate flags."

**B.5. Active Discovery Week (conditional)**
If session-context FLAGS shows a `discovery week LIVE`:
1. Read today's DAY-N file from the path in the FLAGS entry: `Filesystem:read_file → [derived path]`
2. Extract **Day N Observation Targets** + any Q-Partial follow-ups flagged for today
3. Include in briefing as: `📋 [Client] Day N — today's targets: [list]`
Omit entirely if no discovery week is active.

**C. Build Queue (Top 3)**
From BUILD-ROADMAP.md → Next Actions table, show the top 3 *unblocked* items:
- Item number, name, owner, blocked-by status
- Skip items blocked by incomplete prerequisites

**D. Calendar**
If Google Calendar MCP is available → pull today's events.
If NOT available → show: "📅 *Calendar not integrated yet. Check your calendar for today's commitments.*"

**E. Jordan Actions**
From session-context.md → NEXT SESSION → "Jordan actions" list. Show pending items only.

**F. Delta Since Last Session**
From RECENT SESSIONS table — one-line summary of what the most recent session accomplished. Note session number and date.

**G. Model Recommendation**
Check [O]/[S]/[H] tags in NEXT SESSION. Recommend which model to start with:
- All [S]/[H]: "Recommend Sonnet — all ops work today."
- Mix of [S] and [O]: "Recommend Sonnet for N ops items. Opus needed for [specific item] — start there or clear ops first."
- All [O]: "Recommend Opus — strategy work today."
- No tags: default Sonnet.

**H. Weekly L1 Health Check (Monday only)**
If today is Monday (first session after Sunday), run the L1 weekly health summary before presenting the briefing. Query `haios_health_checks` via Safe SQL:
```
curl -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT state, COUNT(*) as count FROM haios_health_checks WHERE timestamp > NOW() - INTERVAL '7 days' GROUP BY state ORDER BY state"}'
```
If any `error` or `warning` rows exist, also pull:
```
SELECT component, state, timestamp FROM haios_health_checks
WHERE state IN ('error','warning') AND timestamp > NOW() - INTERVAL '7 days'
ORDER BY timestamp DESC LIMIT 10
```
Present as a compact block in the briefing:
```
🔧 L1 Weekly Health (past 7 days): {ok: N, warning: N, error: N}
{If any errors/warnings: list component + state + date}
```
If all `ok` and count > 0: "✅ L1 clean." If no rows at all: "⚠️ No L1 health checks in past 7 days — workflows may not be running."

### 3. PROMPT

End with:
```
What first?
```

No menu. No numbered options. Jordan directs.

## Handoff Contract

When complete, state:
- Session number announced
- Briefing delivered (flags, build queue, calendar, Jordan actions, delta)
- Awaiting Jordan's direction

## Dependencies

- Required: Desktop Commander / Filesystem tools (`read_file`)
- Required: `session-context.md`, `BUILD-ROADMAP.md` on Drive
- Optional: Google Calendar MCP (degrades gracefully to manual reminder)

## Authority

Tier 3 (Autonomous) — read-only operations, no external effects

## Error Handling

- **File not found:** Ask Jordan to verify path
- **Desktop Commander unavailable:** Cannot proceed; inform Jordan
- **BUILD-ROADMAP.md missing:** Fall back to session-context.md NEXT SESSION priorities only
- **Malformed file:** Report issue, ask Jordan to check

## Cross-References

- Related: `session-startup-skill` (mid-day lighter reload)
- Related: `session-end-skill` (writes session-context.md this skill reads)
- See also: `BUILD-ROADMAP.md` for build queue source of truth
