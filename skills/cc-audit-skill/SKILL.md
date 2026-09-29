---
name: cc-audit-skill
description: "Audit a completed CC run. Triggers: audit CC run, post-mortem CC, investigate storm, CC failure analysis, storm detected, STORM-DETECTED flag, CC ran bad, compaction storm."
---

# CC Audit Skill

**Version:** 1.0 | **Layer:** AOS

Audit a completed CC run using the reproducible methodology at `HAIOS/Knowledge/CC-RUN-AUDIT-METHODOLOGY.md`. Classifies outcomes (CLEAN / ISSUE / STORM), detects known failure patterns, produces either a brief audit artifact or a full post-mortem. Operational wrapper around the methodology — methodology has the why/how, this skill has the step-by-step.

## Quick Reference

```
GATHER → INPUTS → SCAN signals → CLASSIFY → RECONSTRUCT → ROOT CAUSE → OUTPUT → FOLLOW-UPS
```

## When to invoke

**Automatic triggers** (audit is mandatory):
- `~/.claude/STORM-DETECTED.flag` exists — the `postcompact-log.ps1` hook wrote this after 3+ compactions in 15 min on the same `session_id`.
- CC feedback reports `FAILED` or `PARTIAL-FAILED`.
- Disk check shows CC's claimed output file missing.

**Judgment triggers** (audit recommended):
- Task classified simple but CC reported partial completion.
- Compaction count >3 on a single task.
- Task took >3x budgeted time.
- Jordan flags a run as suspicious.

Full trigger list + skip conditions: methodology §1.

## Workflow

### 1. GATHER — Identify the run
From Jordan or from `session-context.md` / recent dispatched prompts in `HAIOS/Architecture/`:
- Task name + prompt file path
- `session_id` — if storm-flagged, it's in the flag file. Otherwise correlate run timing with compaction log entries.
- Expected outcome (from prompt Definition of Done).

If `session_id` is ambiguous, read the flag file first — it lists the matching `session_id` and recent compaction timestamps.

### 2. INPUTS — Read artifacts (parallel)

Read ALL before diagnosing. Use `Filesystem:read_multiple_files` for parallel reads:

| Artifact | Location | Notes |
|---|---|---|
| CC-PROMPT | `{folder}/CC-PROMPT-{VERB}-{SUBJECT}.md` | The spec |
| CC-FEEDBACK | `{folder}/CC-FEEDBACK-{VERB}-{SUBJECT}.md` | Self-report — do NOT trust uncritically |
| CC-BUILD-LOG | `{folder}/CC-BUILD-LOG-{VERB}-{SUBJECT}.md` | Incremental progress (may be missing on old tasks) |
| Compaction log | `~/.claude/logs\compaction-log.md` | Read tail, grep for `session_id` |
| Storm flag | `~/.claude/STORM-DETECTED.flag` | If exists — has timestamp, session_id, compaction timestamps |
| Settings | `~/.claude/settings.json` | effortLevel, AUTOCOMPACT_PCT, subagent model |
| Output file(s) | Per prompt spec | Ground truth |

Skip missing artifacts but note the absence in the audit output — it's evidence.

### 3. SIGNAL SCAN

Check against the 5 known patterns from methodology §4:

- **Storm** — 3+ compactions in 15 min on same session_id
- [ ] **Judgment-loss** — multi-item classification task + partial completion + no persisted decision table
- [ ] **Safety-gate-held-but-not-completed** — temp writes present, source unchanged, verify step never ran
- [ ] **Verification-theater** — feedback claims verified, no tool-call evidence in build log, disk state contradicts
- [ ] **Compaction-recovery-loop** — prompt predicted "no compaction needed", 3+ observed

Multiple signals can match.

### 4. CLASSIFY

| Classification | Criteria |
|---|---|
| **CLEAN** | Task complete, feedback verified on disk, ≤1 compaction, no signals |
| **ISSUE** | 1 signal matched OR partial completion OR disk-state mismatch, no storm |
| **STORM** | Storm signal matched OR safety-critical disk mismatch OR new anti-pattern candidate |

### 5. RECONSTRUCT — Apply methodology §3

For ISSUE and STORM classifications, execute methodology steps 1-7:
1. **Baseline** — permanent CC tax + path-scoped rules that matched
2. **Task-specific tax** — prompt + target + refs + thinking burn estimate
3. **Timeline reconstruction** — compaction log entries for this session_id, cross-referenced with build log timestamps
4. **Decision state recovery** — what does build log tell us CC knew + when?
5. **Cross-check disk state** — verify every claimed output independently
6. **Signal matching** — done in step 3 above
7. **Root cause** — proximate / systemic / meta-pattern

CLEAN classifications skip steps 1-2 and 7; verify steps 3-5 briefly.

### 6. OUTPUT — Produce audit doc

Follow `doc-management-skill` PRE-FLIGHT (path verified, LAYER-TAXONOMY consulted, new file check).

**Brief audit** (CLEAN or ISSUE):
- Path: `Archive\CC-Prompts\S{session}-{verb}-{subject}-audit.md`
- Format: methodology §5.1
- Length target: 20-50 lines

**Full post-mortem** (STORM or new anti-pattern candidate):
- Path: `HAIOS\Architecture\{TASK}-POST-MORTEM-S{session}.md`
- Format: methodology §5.2 (8 sections)
- Length target: 400-700 lines
- Template: `HAIOS/Architecture/C1-POST-MORTEM.md`

Escalate brief → full if: storm matched, safety-critical disk mismatch, anti-pattern candidate, or >3x budget with partial output.

### 7. FOLLOW-UPS

Surface to Jordan + take Tier-appropriate action per methodology §6:

| Finding | Action | Tier |
|---|---|---|
| New anti-pattern candidate | Draft AP spec; surface for registry add | **Tier 1 approval** |
| Doctrine gap exposed | Identify correct home; draft amendment; apply | **Tier 2 inform after** |
| Tracker row add | Log to `OPUS-4-7-BEST-PRACTICES.md` or relevant tracker | **Tier 3 autonomous** |
| Prompt-side prevention | Update `cc-prompt-skill/references/task-types/{TYPE}.md` + `ANTI-PATTERNS.md`; bump `cc-prompt-skill` | **Tier 2/3** (updates tier-free; version bump prompts Jordan install) |
| Flag file close | Delete `STORM-DETECTED.flag` AFTER audit persisted | **Tier 3** |

## Handoff Contract

When complete, state:
- Audit doc path: `{full path}`
- Classification: `CLEAN` | `ISSUE` | `STORM`
- Signals matched: `{list or "none"}`
- Disk state verified: ✅ all match | ⚠️ `{mismatch detail}`
- Anti-pattern candidate: ✅ new — `{name}` | existing `anti-pattern {N}` | none
- Follow-ups queued: `{count}` — `{brief list}`
- Flag file status (if applicable): ✅ closed | ⚠️ kept open — `{reason}`

## Dependencies

- `Filesystem:read_multiple_files` — parallel artifact collection
- `Filesystem:read_text_file` + head/tail — large compaction log navigation
- `Filesystem:write_file` — audit output
- `HAIOS/Knowledge/CC-RUN-AUDIT-METHODOLOGY.md` — load for signal detail + heuristics (read once per audit)
- `HAIOS/Architecture/C1-POST-MORTEM.md` — template for full post-mortem format
- `doc-management-skill` PRE-FLIGHT — path verification + layer routing

## Authority

- Reading artifacts + producing audit doc: **Tier 3** (autonomous — reversible analytic work)
- Proposing doctrine / template updates: **Tier 2** (inform after)
- Adding to anti-pattern registry: **Tier 1** (approval required)

## Error Handling

| Issue | Action |
|---|---|
| `session_id` unknown, no storm flag | Ask Jordan for prompt file + approximate run time; correlate with compaction log timestamps |
| Compaction log entries suggest different `session_id` than provided | Trust log; ask Jordan to confirm reassignment |
| Output file missing | This IS a finding. CC didn't complete. Classify at least ISSUE, probably STORM |
| Build log missing | Degraded audit — rely on feedback + compaction log. Note the absence explicitly in output |
| Multiple storms in flag file | Audit each cluster separately; one audit per `session_id` + time window |
| Task name unclear from flag | Correlate flag timestamp with dispatched prompts in `HAIOS/Architecture/` by filename + mtime |

## Cross-References

- **Methodology:** `HAIOS/Knowledge/CC-RUN-AUDIT-METHODOLOGY.md` (authoritative — this skill is the wrapper)
- **Worked example:** `HAIOS/Architecture/C1-POST-MORTEM.md`
- **Storm detection:** `~/.claude/hooks/postcompact-log.ps1` (writes the flag)
- **Anti-pattern:** "CC Self-Refactor Storm" (first pattern this methodology codified)
- **Related skill:** `cc-prompt-skill` (updates flow back into prompts / templates / anti-patterns)
- **Automation target:** methodology §8 (scheduled n8n workflow after 3-5 validation runs)

## Notes

- **This is a COO skill, not a CC skill.** CC produces the raw material (build logs, feedback, compaction log entries). COO audits. Separation of concerns.
- **Flag presence = open audit.** Don't delete `STORM-DETECTED.flag` until the audit artifact is persisted. If Jordan sees the flag, there's a live audit item.
- **Disk state is the oracle.** When feedback and disk disagree, disk wins. Audit the discrepancy, not the feedback text.
- **Feed the audits back.** Every audit is raw material for future audits. Look at prior `-audit.md` files in `Archive/CC-Prompts/` before concluding "new pattern" — existing pattern re-observed is different action than new pattern.
