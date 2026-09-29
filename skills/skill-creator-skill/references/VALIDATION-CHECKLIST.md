# Skill Validation Checklist

Deterministic checks before a skill is complete. All must pass.

---

## Required Sections (Schema Check)

Run mentally or grep for these headings:

| Section | Required? | Present? |
|---|---|---|
| Frontmatter (`---` block) | ✅ | [ ] |
| `name:` in frontmatter | ✅ | [ ] |
| `description:` with "Use when:" | ✅ | [ ] |
| `## Quick Reference` | ✅ | [ ] |
| `## Workflow` | ✅ | [ ] |
| `## Handoff Contract` | ✅ | [ ] |
| `## Dependencies` | ✅ | [ ] |
| `## Authority` | ✅ | [ ] |
| `## Error Handling` | ✅ | [ ] |

**Pass criteria:** All required sections present.

---

## Token Budget (Size Check)

Estimate or count tokens:

| File | Tokens | Budget | Pass? |
|---|---|---|---|
| SKILL.md | [count] | <1,000 | [ ] |
| references/* total | [count] | <3,000 | [ ] |
| Entire skill | [count] | <5,000 | [ ] |

**Counting method:** Word count × 4 = approximate tokens

**Pass criteria:** All under budget, or Jordan approved exception.

---

## Trigger Uniqueness (Conflict Check)

1. Extract triggers from new skill's description
2. Search existing skill descriptions in `claude-instructions.md`
3. Flag any overlap

| New Trigger | Conflicting Skill? | Resolution |
|---|---|---|
| [trigger 1] | [skill or "none"] | [ ] |
| [trigger 2] | [skill or "none"] | [ ] |

**Pass criteria:** No unresolved conflicts.

---

## Authority Appropriateness (Risk Check)

| Skill Actions | Appropriate Tier |
|---|---|
| Read-only (search, view) | Tier 3 |
| Internal writes (docs, configs) | Tier 2-3 |
| External effects (publish, send) | Tier 1-2 |
| Spend, delete production, bypass compliance | Tier 0 (forbidden) |

**Declared tier:** [tier]
**Highest-risk action in skill:** [action]
**Match?** [ ]

**Pass criteria:** Tier matches or exceeds risk level.

---

## Error Handling Coverage (Completeness Check)

List likely failure modes for this skill type:

| Failure Mode | Handled? |
|---|---|
| Required file not found | [ ] |
| Tool/API unavailable | [ ] |
| Invalid input from user | [ ] |
| Partial completion | [ ] |
| Unknown/unexpected error | [ ] |

**Pass criteria:** All likely failures have handling defined.

---

## Self-Verification Questions

Answer before completing:

1. **Requirement trace:** Can I map each of Jordan's requirements to specific skill content?
   - [ ] Yes / [ ] No — gap: ___

2. **Cold start test:** Would a fresh Claude session (no prior context) understand how to use this skill?
   - [ ] Yes / [ ] No — unclear part: ___

3. **Outcome clarity:** Is success/failure unambiguous?
   - [ ] Yes / [ ] No — ambiguity: ___

**Pass criteria:** All "Yes" or gaps resolved.

---

## Final Status

| Check | Pass? |
|---|---|
| Required Sections | [ ] |
| Token Budget | [ ] |
| Trigger Unique | [ ] |
| Authority Appropriate | [ ] |
| Errors Handled | [ ] |
| Self-Verified | [ ] |

**Overall:** [ ] PASS — ready for backup & package / [ ] FAIL — resolve issues first
