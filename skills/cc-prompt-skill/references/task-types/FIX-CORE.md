# FIX Task Type

Bugs, broken configs, failing tests.

**Recommended effort:** `high`

- Override to `xhigh` for root-cause hunts, unclear reproduction, or deep debugging
- Override to `medium` for known-bug patch with tight scope

<!-- COO uses this file when writing a CC prompt for a FIX task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## BUG
- **Symptom:** {what's happening}
- **Expected:** {what should happen}
- **Reproduction:** {exact input, execution ID, error message from most recent failure}
- **Suspected cause:** {if known — helps CC start in the right place}
- **Files involved:** {with full paths}
- **Trigger method:** `webhook` | `jordan_ui` — schedule-triggered workflows cannot be triggered via REST API (POST /run → 405, Pitfall #13). If `jordan_ui`: CC uses GET /executions with includeData=true for REPRODUCE-FIRST diagnosis; L1 manual trigger requires Jordan to fire from n8n UI.

## REPRODUCE-FIRST (mandatory for workflow fixes)
Per `.claude\rules\workflow-verification.md`, CC must reproduce the original failure BEFORE applying any fix:
1. Use exact input/error from Reproduction section above
2. Reproduce via webhook test; or for `jordan_ui` (schedule-triggered) workflows: use GET /executions with includeData=true on known-bad execution IDs — no REST execute available (Pitfall #13)
3. Document in build log: input used, execution ID, error observed, timestamp
4. Only then proceed to fix — "fixed" means this reproduction no longer fails

**Dismissed-TTL caveat:** If the workflow uses `haios_remediation_log` dedup with a 30-day dismissed TTL, a post-fix trigger run on the SAME DAY as dismissals will NOT produce new DB inserts for the dismissed bugs -- TTL blocks re-insertion. Note "L2 DB verification deferred to next undismissed-state run" in build log; verify same-day by inspecting execution data (e.g., `allFindings` array in Compile Report output -- findings count for the fixed check should be 0).

## VERIFICATION (4-layer standard)
Per `.claude\rules\workflow-verification.md`, all 4 layers required:
- **L1:** HC state='ok' exists with timestamp AFTER fix applied
- **L2:** Original failure no longer reproduces with same input from REPRODUCE-FIRST
- **L3:** Correct data in output table (task-specific query)
- **L4:** COO independent check — CC leaves L4 blank in feedback file
{Include task-specific verification queries for L1 and L3. L2 is automatic from reproduce-first step.}
```

## Fix-Target Scoping — syntax/tokenizer-class root causes

When the FIX's root cause is an **expression-syntax / tokenizer / JSON-parse class** bug (not a data or config bug specific to one node), do NOT scope the fix to the single node the reproduction surfaced. Before finalizing the fix-target list, **grep the WHOLE workflow's serialized JSON** for the same syntactic pattern — the broken expression shape, `cache_control`, `JSON.stringify({` nesting depth, etc. — and **add that whole-workflow grep as a §5 CONTEXT QUERY** in the prompt. A shared root cause from a prior workflow-wide restructure (e.g. an M3-style body change) lives on every node it touched. One execution proves ONE node broke on ONE run; it does NOT prove only one node carries the pattern. *(Origin: FIX-ANALYZER-JSONBODY — DoD named only `3P Classifier API`; the identical `cache_control` break on unscoped sibling `Claude API` surfaced on first live fire, costing an extra Jordan-fired round-trip.)*

## Schedule-Triggered Verification — CC cannot self-trigger the post-fix run

For any FIX or BUILD on a **schedule-triggered** n8n workflow, do NOT assume CC can fire its own post-fix verification run. Two mechanisms compound: (1) n8n MCP auth is unreliable in CC sessions, and (2) schedule-trigger workflows have no REST manual-run path (`POST /run` → 405, Pitfall #13). CC can GET/PUT via REST, but the run that proves the fix works must be **Jordan-fired from the n8n UI** or the next natural scheduled fire. Plan that path at authoring time so CC applies the fix and stops cleanly instead of spinning to self-trigger:

- **§1 CC ACCESS CHECK bullet:** "Confirm n8n MCP auth. If unavailable, use REST for GET/PUT and treat manual-run verification as Jordan-dependent."
- **EXECUTION POLICY fallback:** "If no manual-run path is available to CC, apply the fix + PUT, then STOP and request ONE Jordan-fired manual run from the n8n UI."
- **`/goal` anchoring:** anchor to "fix applied + PUT structurally verified," with L1/L2 explicitly gated on the Jordan-fired or next-scheduled run — so CC does not treat "self-triggered a run" as part of done.

*(Origin: FIX-QUARANTINE-HEARTBEAT / BUILD-WEEKLY-AUDIT-CHECK8 + FIX-QM-EMPTY-BRANCH — both deferred L1/L2 "pending next trigger"; the empty-branch 0-row run had to be Jordan-fired, exec 44947. n=2.)* Cross-ref: `N8N-API-REFERENCE.md` Pitfall #13.

## Verification Subagent (mandatory for workflow FIX tasks)

Do NOT run POST-PUT GET verification in main context.
Spawn verification subagent (Haiku):
"GET workflow {id}. Check assertions: [L1 checklist from DoD].
Return: PASS/FAIL per assertion, node count, availableInMCP."
Subagent holds the 50KB JSON. Main session receives assertion table only.

---

## Session Learnings

Accumulated learnings moved to `FIX-LEARNINGS.md` (COO-only reference). CC does NOT load that file at runtime -- curated learnings arrive in the prompt's §8 IMPORTANT NOTES when COO selects them during prompt authoring.
