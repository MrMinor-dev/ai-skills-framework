# Install Workflow

> **Authoritative pipeline lives in SKILL.md (CORE RULE + Steps 4–7).** This file expands the install and post-install detail.

Skills require a human-in-the-loop install process. Claude packages, Jordan installs.

---

## Why Human Required

**Tier 0H (Human-Required Technical Limitation):**
- Claude's container resets each session
- Only Jordan can modify Claude Desktop settings
- Skill installation persists across sessions

---

## Workflow

### Step 1: Package

The skill is authored **in the sandbox** (`/home/claude/{skill-name}/`) during Step 4 of the SKILL.md pipeline, so packaging is a straight zip — no bridge required:

```bash
cd /home/claude
rm -f /mnt/user-data/outputs/{skill-name}.skill
zip -r -X /mnt/user-data/outputs/{skill-name}.skill {skill-name}/
unzip -t /mnt/user-data/outputs/{skill-name}.skill   # must report no errors
```

This creates `{skill-name}.skill` (a zip with `.skill` extension) directly in the outputs directory.

### Step 2: Present to Jordan

Use `present_files` tool to make `.skill` file downloadable.

State clearly:
```
📦 Skill packaged: [skill-name]
- Download: [link]
- Install location: Claude Desktop → Settings → Skills → Add Custom Skill
- After install: Start new session to use
```

### Step 3: Sync to Drive

After presenting, write the canonical files to Drive (SSOT):

| Layer | Path |
|---|---|
| HAIOS | `HAIOS\Skills\[skill-name]\` |
| AOS | `AOS\Skills\[skill-name]\` |
| PB | `PB\Skills\[skill-name]\` |

Write only changed files. Unchanged reference files don't need a re-write.

**Folder structure:** Each skill gets its own folder:
```
Skills/
├── SKILLS-ROADMAP.md
├── SKILL-ORGANIZATION.md
├── skill-name/           ← CORRECT (folder per skill)
│   ├── SKILL.md
│   └── references/
└── old-skill.md          ← LEGACY (migrate to folder)
```

**Version existing:** If updating a skill, the folder already exists — just update the changed files.
**Legacy flat files:** When refactoring a skill that exists as a single `.md` file, create the folder structure and delete the flat file.

### Step 4: Update SKILLS-ROADMAP.md

Change status to 📦 Packaged:
```markdown
| `skill-name` | NEW | 📦 Packaged | 192 | Awaiting Jordan install |
```

### Step 5: Jordan Installs

Jordan's actions (0H - human required):
1. Download `.skill` file from Claude's output
2. Open Claude Desktop → Settings → Skills
3. Click "Add Custom Skill" or drag-drop
4. Confirm install
5. Start new conversation to activate

### Step 6: Post-Install (Next Session)

In the next session after Jordan confirms install:

1. **Update SKILLS-ROADMAP.md** → ✅ Installed

2. **Update SKILL-ORGANIZATION.md** (if new skill):
   - Add to appropriate layer table
   - Include triggers

3. **Update `HAIOS/claude-instructions.md`**:
   - Add skill to 🧠 SKILLS table (if new) or update triggers
   - Bump VERSION number
   - Update LAST UPDATED session number
   - Note: This is the SSOT that feeds userPreferences

4. **Prompt Jordan**:
   ```
   Please update userPreferences in Claude Desktop:
   Settings → Profile → Copy content from HAIOS/claude-instructions.md
   ```

**Note:** Index updates are orchestrated separately — batch with other doc changes at session end.

---

## Status Flow

```
❌ Not started
    ↓ (Claude builds in sandbox)
🔨 In progress
    ↓ (Claude zips + present_files)
📦 Packaged (awaiting Jordan install)
    ↓ (Jordan installs in Claude Desktop)
✅ Installed
    ↓ (Claude updates docs, Jordan updates userPreferences)
🔄 Synced (claude-instructions.md = userPreferences)
```

---

## Tracking Pending Installs

SKILLS-ROADMAP.md tracks all skill statuses. Filter for 📦 to see pending:

```markdown
| Skill | Status | Notes |
|---|---|---|
| `skill-creator-skill` | 📦 Packaged | Awaiting Jordan install |
```

Jordan can batch-install multiple skills at once.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Skill not appearing after install | Restart Claude Desktop, start new chat |
| zip/unzip -t fails | Check SKILL.md frontmatter format; rebuild sandbox tree from Drive canonical |
| Skill triggers but errors | Check dependencies, paths |
| Backup location wrong | Verify layer placement in SKILL-ORGANIZATION.md |
| userPreferences out of sync | Copy claude-instructions.md content to Settings → Profile |
