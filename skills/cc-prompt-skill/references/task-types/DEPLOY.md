# DEPLOY Task Type

Activate, push, release.

**Recommended effort:** `low`

- Override to `medium` when deploy involves config decisions or rollback planning
- Rarely exceeds `medium` — if it does, scope creep suggests a BUILD

<!-- COO uses this file when writing a CC prompt for a DEPLOY task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## DEPLOY STEPS
{Ordered steps. Include pre-deploy checks and post-deploy verification.}

## ROLLBACK
{What to do if deploy fails. Specific commands to undo.}
```

## Session Learnings

_No session-accumulated learnings yet._
