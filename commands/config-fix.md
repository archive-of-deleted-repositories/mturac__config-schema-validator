# /config-fix

**Trigger**: `/config-fix [file]`

**Purpose**: Interactively fix config issues with before/after diffs, per-fix confirmation, and automatic backup.

**Arguments**:
- `[file]` (optional): path to a specific config file to fix. If omitted, fixes all files with issues.

**Behavior**:
- Invokes the `skills/config-fix` skill
- Backs up each file to `.config-backup/` before any edits
- Shows before/after diff for each fix
- Asks `[y/N/skip-file/quit]` per fix
- Only ERRORs and WARNs are offered for fixing (INFO is skipped)

**Example session**:
```
─────────────────────────────────────
[ERROR] docker-compose.yml:8
Issue: Service 'worker' has neither 'image' nor 'build'

Before:
  worker:
    ports:
      - "5000:5000"

After:
  worker:
    image: placeholder:latest  # TODO: set your image
    ports:
      - "5000:5000"

Apply this fix? [y/N/skip-file/quit] y
✓ Fixed — Backed up to .config-backup/docker-compose.yml.bak
─────────────────────────────────────
Fix session complete:
  Applied:  1 fix
  Skipped:  0
  Manual:   0
```
