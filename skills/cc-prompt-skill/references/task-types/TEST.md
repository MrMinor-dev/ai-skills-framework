# TEST Task Type

Run tests, verify behavior.

**Recommended effort:** `medium`

- Override to `xhigh` for novel test logic or coverage design
- Override to `low` for boilerplate test copy or simple assertion runs

<!-- COO uses this file when writing a CC prompt for a TEST task. Copy the spec block below into the prompt's §7 TASK-TYPE section, then embed relevant session-learning notes in §8 IMPORTANT NOTES. -->

## Spec Block

```markdown
## TEST PLAN
{What to test, test payloads, expected outputs per test case.
Include Known Limitations per test — what each test CAN and CANNOT verify.}

## VERIFICATION QUERIES
{SQL queries, curl commands, or checks CC runs to confirm results.}
```

## Session Learnings

**Workflow TEST notes:**
- Multi-trigger workflows: MCP `execute_workflow` fires the **first trigger in node order** (typically Webhook), NOT the Schedule trigger. If testing the schedule path, check the execution's `mode` field after running. If `mode: "webhook"`, the wrong path fired. **Fallback:** use safe-sql to verify the detection SQL directly and document as PARTIAL — "SQL logic verified; schedule path E2E pending next cron run." See ANTI-PATTERNS #43.
- Confirm expected trigger path in the prompt: if the workflow has both Webhook and Schedule triggers, state explicitly which path is under test and how to verify it ran.
