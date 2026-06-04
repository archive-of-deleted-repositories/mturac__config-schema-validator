# config-fix

Apply fixes interactively for config file issues detected by config-scan.

## Trigger

`/config-fix [file]` or when user asks to fix, repair, or auto-fix config issues.

If `[file]` is provided, operate only on that file.
If no file provided, run config-scan first, then fix all files with issues.

## Steps

### 1. Collect Issues

Run config-scan skill first (or reuse results from current session if available).
Filter issues to only `ERROR` and `WARN` severity — skip `INFO`.

If a specific file was given: filter issues to that file only.

### 2. Backup Before Touching

Before modifying any file, create a backup:

```bash
mkdir -p .config-backup
cp <file> .config-backup/<filename>.bak
```

Inform user: `Backed up <file> → .config-backup/<filename>.bak`

Only backup once per file per session (don't re-backup if already backed up this run).

### 3. Present Each Fix Interactively

For each issue (ordered: ERRORs first, then WARNs):

Show:
```
─────────────────────────────────────
[ERROR] docker-compose.yml:12
Issue: Service 'api' has neither 'image' nor 'build'

Before:
  api:
    ports:
      - "8080:8080"

After:
  api:
    image: your-image:latest   # ← add this
    ports:
      - "8080:8080"

Apply this fix? [y/N/skip-file/quit]
─────────────────────────────────────
```

Wait for user input:
- `y` or `yes` → apply fix
- `n` or `no` or Enter → skip this fix
- `skip-file` → skip all remaining fixes for this file
- `quit` or `q` → stop all fixing, show summary

### 4. Apply Confirmed Fixes

When user confirms (`y`):
1. Read the file content
2. Apply the minimal change (line replacement or insertion)
3. Write the file back
4. Print: `✓ Fixed: <description>`

When skipped:
- Print: `— Skipped`

### 5. Fix Catalog

Known fix transformations:

| Issue | Fix |
|---|---|
| Tab indentation in YAML | Replace leading `\t` with 2 spaces on each affected line |
| Missing `image`/`build` in docker-compose service | Insert `image: placeholder:latest  # TODO: set your image` after service key |
| GitHub Actions step missing `run`/`uses` | Insert `run: echo "TODO: add command"` |
| `"on"` written as `on` (bare) in YAML top-level | Wrap key in quotes: `"on":` |
| `latest` image tag | Add inline comment: `# WARNING: pin this tag` |
| Missing `apiVersion` in Kubernetes | Insert `apiVersion: apps/v1` at top of file |
| JSON parse error | Cannot auto-fix — show error location, ask user to fix manually |
| Duplicate keys | Cannot auto-fix — highlight both occurrences, ask user to resolve |

For issues without an auto-fix: display `Cannot auto-fix: <reason>. Manual fix required.` and skip.

### 6. Summary

After all fixes processed:

```
─────────────────────────────────────
Fix session complete:
  Applied:  X fixes
  Skipped:  Y fixes
  Manual:   Z fixes need manual attention

Backups saved in: .config-backup/
─────────────────────────────────────
```
