---
name: workflow-build-skill
description: "Build n8n workflows. Triggers: 'build workflow', 'create workflow', 'new workflow'."
updated: 2026-04-18
---

# Workflow Build

> **REFERENCE ONLY — Superseded.** COO no longer builds workflows. CC owns workflow building via the n8n-workflow-build CC skill + n8n-workflow-builder agent. This doc retained as reference. Do not follow as active procedure.

**Version:** 2.0

5-phase gated process. Load only the reference for the active phase — never load all at once.

## Phases

| # | Phase | Gate (don't advance until met) | Reference |
|---|---|---|---|
| 1 | **SPEC** | Jordan approves spec | `references/SPEC-CHECKLIST.md` |
| 2 | **SCAFFOLD** | Workflow exists in n8n, nodes wired, validated | `references/BUILD-PATTERNS.md` |
| 3 | **TEST** | ONE input → correct output verified in DB | `references/TEST-AND-DEBUG.md` |
| 4 | **DEBUG** | All bugs from test resolved, re-verified | `references/TEST-AND-DEBUG.md` |
| 5 | **DEPLOY** | Definition of Done checklist passes | `references/DEPLOY-CHECKLIST.md` |

**Workflow:** Read the reference for your current phase → execute → confirm gate → advance.

## Build Behavior

- **Work silently during builds.** Report outcomes, not process steps. Exception: decisions requiring Jordan input.
- **Credential lookup:** Check `session-context.md` REFERENCE section first. Semantic search second. Loading full workflows for credential IDs is a last resort.

## Token Management

### Budget Awareness
- **Before starting any phase:** Check context usage. If >60% consumed, checkpoint before building.
- **Phase 2 WARNING:** Phase 2 is the heaviest context consumer (node research + credential lookup + workflow JSON). If approaching 60% context at Phase 2 start, save README BUILD STATE before proceeding.
- **After Phase 2 (Scaffold):** Natural checkpoint. If >50% consumed, save state and hand off.
- **Phases 3-4 (Test/Debug) are the most token-intensive.** They require loading execution data, comparing shapes, iterating. Start these with at least 50% context remaining.
- **Never start a new phase after compaction.** Compaction loses execution context, node output shapes, and intermediate findings. These cannot be reconstructed from docs alone.

### Compaction Rules
If context pressure hits ~75% mid-build:
1. **Stop building immediately** — do not start another test/debug cycle
2. Run the checkpoint procedure (below)
3. Tell Jordan: "Context at ~75%. Checkpointed at Phase [N]. Next session continues from here."
4. Do NOT attempt to "squeeze in one more fix" — this is how we lose work

## Session Handoff

### Checkpoint Procedure
When a build spans multiple sessions (planned or forced by context):

**1. Update the workflow README** with current ACTUAL state:
```markdown
## BUILD STATE (Session NNN)
- **Phase:** [1-5] — [phase name]
- **Gate Status:** [PASS / IN PROGRESS / BLOCKED]
- **Workflow ID:** [id or TBD]
- **What works:** [specific: "Trigger → Fetch → Normalize chain tested with r/dogs, 3 rows written"]
- **What's broken:** [specific: "Expression in Upsert node references $json.title but upstream returns $json.data.children[0].title"]  
- **Next action:** [specific: "Fix Upsert expression, re-test with r/dogs"]
- **Execution IDs to review:** [list any relevant execution IDs]
```

**2. Update `session-context.md`** with carry-forward:
- Current phase + gate status
- Workflow ID
- Specific next action (not vague — exact node, exact fix)

**3. Next session opening move:**
When resuming a build, the COO should:
1. Read the workflow README (actual state, not the plan sections)
2. Read the workflow from n8n: `n8n_get_workflow → id`
3. Check last execution: `n8n_list_executions → workflowId, limit: 1`
4. Load only the reference for the current phase
5. Resume from the specific next action in the checkpoint

### Phase Boundaries = Natural Handoff Points
- **End of Phase 1 (Spec):** Cheapest handoff. Spec is in the README, no n8n state to lose.
- **End of Phase 2 (Scaffold):** Good handoff. Workflow exists in n8n, can be re-read.
- **Mid Phase 3-4 (Test/Debug):** Expensive handoff. Execution context and intermediate findings are lost. Write them into the README BUILD STATE section before stopping.
- **Phase 5 (Deploy):** Checklist-driven, easy to resume.

## Anti-Patterns (Proven Session-Burners)

| Anti-Pattern | What Happened | Rule |
|---|---|---|
| Build full pipeline before testing one node | Invoice Handler | Phase 3 gate: ONE input first |
| Trust "ran successfully" without checking DB | Invoice Handler | Verify destination state with SQL |
| Use `n8n_update_partial_workflow` | Failed across multiple workflows | Always `n8n_update_full_workflow` |
| Debug by guessing instead of reading execution data | Email Router | Phase 4: execution forensics FIRST |
| Assume node output shape | Invoice Handler | Only execution data reveals truth |
| Fix multiple bugs in one update | Email Router | One bug per cycle, re-test between |
| Continue building after compaction | Lost context, re-guessed bugs | Checkpoint and hand off instead |
| Load full workflows for credential IDs | Reddit Ingest (~30K tokens) | Semantic search first (~500 tokens) |
| Add Slack/notification nodes to operational workflows | Reddit Ingest — removed by Jordan | Producers write DB, monitors read DB |

## Handoff Contract

After each phase, state:
- Phase completed: [N]
- Gate: PASS / FAIL
- What was verified (specifically)
- Next phase ready: YES / NO + blockers
- Context budget: approximate % remaining

After Phase 5:
- Workflow: [name] ([ID])
- Status: ACTIVE / PENDING (reason)
- DoD: all checked
- Files updated: [paths]

## Dependencies

- n8n MCP + n8n-mcp: workflow CRUD, node docs, validation, execution inspection
- Database skill: schema verification + output checks
- Desktop Commander: README and doc file operations
- Doc management skill: doc updates in Phase 5

## Authority

- Phases 1-4: Tier 3 (autonomous)
- Phase 5 activate: Tier 2 (inform after)
- Spend/external: Tier 1 (approval required)

## Error Handling

- **n8n API unreachable:** Cannot proceed. Inform Jordan.
- **Node type not found:** `n8n-mcp:search_nodes`. May need different name.
- **Validation errors:** Fix before proceeding — never deploy broken workflows.
- **MCP access lost:** Jordan re-toggles in n8n. Cannot edit until restored.
- **Context >75%:** Checkpoint immediately. Do not start new build cycles.
