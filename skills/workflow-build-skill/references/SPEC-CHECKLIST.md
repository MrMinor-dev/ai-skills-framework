# Phase 1: SPEC Checklist

**Gate:** Jordan approves spec before any n8n work begins.

## 1.1 Gather Context
- Check `AOS/Domains/SERVICE-CATALOG.md` — does this capability already exist?
- Check existing workflow READMEs in `[Layer]/Workflows/` — reusable patterns?
- If README exists → treat as PLAN not FACT until built and verified

## 1.2 Classify (Build Doctrine)
- **AOS or niche-specific?** Default AOS. Uncertain → ask Jordan.
- **Parameterized?** Must accept `business_id` if AOS-level.

## 1.3 Schema Check
Query actual DB — don't trust docs alone:
```bash
curl -s -X POST https://{{N8N_HOST}}/webhook/{{WEBHOOK_ID}} \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT column_name, data_type, is_nullable FROM information_schema.columns WHERE table_name = '\''TABLE_NAME'\'' ORDER BY ordinal_position"}'
```
Also check constraints:
```sql
SELECT constraint_name, check_clause FROM information_schema.check_constraints 
WHERE constraint_schema = 'public' AND constraint_name LIKE '%TABLE_NAME%'
```
Verify: tables exist, column types match, constraints won't reject your data.

## 1.4 Spec Output
Present to Jordan for approval:
- **Pipeline diagram** — text flowchart showing node chain
- **Input → Output shape** — what data enters, what gets written where
- **Schema tables + operations** — READ vs WRITE, key columns
- **Error approach** — what happens when external calls fail
- **Success criteria** — specific and measurable (e.g., "N rows in table X with columns Y, Z filled")

## Phase 1 Complete When:
- [ ] Capability doesn't already exist (or extending existing)
- [ ] AOS vs niche classification decided
- [ ] Schema verified against actual DB
- [ ] Spec presented and Jordan approved
- [ ] README created/updated with spec (but marked as PLANNED)
