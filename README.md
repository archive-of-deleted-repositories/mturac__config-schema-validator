# config-schema-validator

Validate app config files (YAML, TOML, JSON) against schemas and suggest auto-fixes for common misconfigurations.

## Install

```bash
claude plugin install config-schema-validator
```

## Commands

| Command | Description |
|---|---|
| `/config-validate` | Scan all config files for issues |
| `/config-fix [file]` | Fix issues interactively with diffs and confirmation |
| `/config-schema [file]` | Infer and save a JSON Schema from a config file |

## Supported Config Types

- **Docker Compose** — validates services, ports, depends_on
- **GitHub Actions** — validates jobs, steps, triggers
- **Kubernetes** — validates apiVersion, kind, metadata, spec
- **pyproject.toml** — validates build-system, project metadata
- **Cargo.toml** — validates package, edition, dependencies
- **ESLint** — validates rules format, extends
- **Jest** — validates testEnvironment, transform
- **Generic JSON/YAML** — catches parse errors, tabs, duplicate keys

## Example Validation Output

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

## Custom Schema Inference

Generate a schema from any config file:

```
/config-schema myapp.config.json
```

Saves to `.config-schemas/myapp.config.json.schema.json` and is used automatically in future `/config-validate` runs.

## Backups

`/config-fix` always backs up files to `.config-backup/` before modifying them.
