# /config-validate

**Trigger**: `/config-validate`

**Purpose**: Scan all config files in the project and report validation issues.

**Behavior**:
- Invokes the `skills/config-scan` skill
- No arguments needed — scans the entire project
- Outputs grouped issue list by file, then summary

**Example output**:
```
Scanning config files...

[ERROR] docker-compose.yml:8
  Type: missing-required-field
  Issue: Service 'worker' has neither 'image' nor 'build'
  Fix: Add 'image: your-image:tag' or 'build: .' under 'worker'

[WARN] .github/workflows/ci.yml:34
  Type: deprecated-action
  Issue: 'actions/checkout@v1' is deprecated
  Fix: Update to 'actions/checkout@v4'

[ERROR] pyproject.toml:3
  Type: missing-required-field
  Issue: [build-system] missing 'build-backend'
  Fix: Add 'build-backend = "setuptools.build_meta"'

─────────────────────────────────
Scanned 6 files — 2 errors, 1 warning
Run /config-fix to apply fixes interactively.
```
