# CREATE Task Type

Files, templates, configs, docs.

**Recommended effort:** `medium`

- Override to `xhigh` for original framework, taxonomy, or doctrine design
- Override to `low` for doc stubs, boilerplate fill-in, or template instantiation
- Override to `off` for pure classification/extraction/tagging (explicit opt-in)

<!-- COO uses this file when writing a CC prompt for a CREATE task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## DELIVERABLE
- **Output:** {file path + format}
- **Content spec:** {what it should contain}
- **Standards:** {style guide, conventions, templates to follow}
```

## Session Learnings

**Output path:** Always specify the full output path for deliverables, including the directory. "Place in output directory" without a path forces CC to infer the location -- it will choose reasonably but may not match COO intent. Use the same specificity as CONTEXT file paths: `AOS\Skills\doc-management-skill.skill`, not "output directory for Jordan."

**For meta-skills (skills that process CC's own output):** use the 4-phase pattern: Read → Classify → Draft → Output. Extract classification/reference tables to `references/` to stay under 120 lines. Reference implementation: `~/.claude/skills/process-feedback/`.
