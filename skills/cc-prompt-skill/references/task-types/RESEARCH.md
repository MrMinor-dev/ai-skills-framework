# RESEARCH Task Type

Investigate, compare, recommend.

**Recommended effort:** `xhigh`

Class default — synthesis, trade-off evaluation, open-ended analysis. If a "research" task can be answered by single-source lookup, it's not a research task. Rarely lowered.

<!-- COO uses this file when writing a CC prompt for a RESEARCH task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## QUESTIONS
{Numbered list of specific questions to answer.}

## COMPARISON CRITERIA
{If comparing options: what dimensions matter, in priority order.}

## OUTPUT FRAMING
- **Audience:** {who reads this — Jordan / team / external}
- **Decision:** {what decision this informs — approval / prioritization / design choice}
- **Format:** {markdown doc / table / one-pager / bullets}
- **Length:** {target word count or page count — e.g., "1-2 pages max"}
- **File path:** {where to write findings}
```

## Session Learnings

**RESEARCH task notes — large doc-tree scans:**
- **Archive/ policy:** Unless the prompt explicitly lists `Archive/` as a target, skip it entirely. Archived = intentionally retired content. This is Jordan's standing preference. Do not add Archive/ to scan scope without explicit instruction.
- **Large recursive scans (200+ files):** Prefer explore agents over grep-filter-then-read. `grep --exclude-dir` with `--include` is unreliable on Windows Git Bash. For research tasks spanning hundreds of files, launch 2-3 parallel explore agents with explicit file lists rather than grep-first, read-second.
- **"N domain roadmaps" vs. master index:** If BUILD-ROADMAP.md (or equivalent master index) is listed as one of the 6 setup files but the task refers to "5 domain roadmaps," state the distinction explicitly. Master index != peer domain roadmap.
- **"Uncaptured" definition edge cases:** Specify whether a one-line domain roadmap placeholder counts as "captured" or whether a full spec elsewhere is still "uncaptured context." Without this, CC must infer and may over- or under-report.
- **Scope of high-volume non-operational dirs (e.g., PB/):** If a large directory is job-search, portfolio, or otherwise non-operational in nature, note it in the prompt (include or exclude). CC will otherwise spend time assessing its relevance.
