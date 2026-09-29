# CC Feedback Protocol

**Append this entire section to every CC prompt file.** CC reads it and writes the feedback file as its final action before reporting done.

---

*Copy everything below this line into the CC prompt file:*

---

## FEEDBACK PROTOCOL

**Before reporting done, write a feedback file.** This is how we continuously improve CC task delegation.

**File:** Save to the same folder as this prompt file, named `CC-FEEDBACK-{same VERB-SUBJECT as prompt}.md`

**Then archive the PROMPT file only (leave feedback for COO review):**
```bash
# Archive prompt file (create dir if needed)
mkdir -p "Archive/CC-Prompts"
mv "{this prompt file path}" "Archive/CC-Prompts/S{session from prompt}-{verb}-{subject}-prompt.md"

# DO NOT archive the feedback file — COO will review and archive it after processing learnings.
```

**Feedback file contents — fill in every section:**

```markdown
# CC Feedback: {task name}
**Date:** {today}
**Prompt file:** {original path}
**Outcome:** SUCCESS | PARTIAL | BLOCKED
**Duration:** {approximate time from start to finish}

## Prompt Quality

### What worked well
- {Specific things in the prompt that made execution smooth}
- {Good context, clear specs, helpful examples, etc.}

### What was ambiguous or missing
- {Things CC had to guess or figure out independently}
- {Missing file paths, unclear specs, assumed context}
- {Information that would have saved time if included}

### Prompt template recommendations
- {How should the prompt template itself be improved?}
- {Missing sections, unnecessary sections, better ordering}

### Instruction hierarchy check
- {Did any IMPORTANT NOTES or inline instructions duplicate content already in `~/.claude/rules/`, `~/.claude/skills/`, or `~/.claude/CLAUDE.md`? List them.}
- {Did any instruction appear for the first time that should migrate UP to a higher-reliability layer? Recommend where: rule, skill, or CLAUDE.md.}

## Environment Recommendations

Rate each area: ✅ No changes needed | 🔧 Improvement suggested | ➕ New addition needed

### Global CLAUDE.md (~/.claude/CLAUDE.md)
{Status + specific recommendation if any}

### Project CLAUDE.md
{Status + specific recommendation if any}

### Hooks (.claude/settings.json)
{Status + specific recommendation. Are there commands that should be blocked? Verified? Automated?}

### Skills (~/.claude/skills/)
{Status + specific recommendation. Missing skill? Skill that needs updating? Skill that was wrong?}

### Tools / MCP servers
{Status + specific recommendation. Missing tool? Tool that didn't work as expected?}

### .claude/rules/ files
{Status + specific recommendation. New rule needed? Existing rule that caused issues?}

### Secrets (.secrets/)
{Status + specific recommendation. New key needed? Key that was missing?}

## Audit Self-Score (if audit criteria were embedded in prompt)
- **Expected:** {X}/{total}
- **Actual:** {X}/{total}
- **Gaps:** {which criteria fell short and why}
- **Skip if no audit criteria were provided**

## Task-Specific Notes
{Anything else the COO should know — edge cases discovered, decisions CC made autonomously, things that need human follow-up}
```
