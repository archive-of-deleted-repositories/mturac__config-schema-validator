# config-scan

Scan project config files and validate them against known schemas.

## Trigger

`/config-validate` or when user asks to validate, check, or scan config files.

## Steps

### 1. Discover Config Files

Use Bash to find all config files, excluding noise:

```bash
find . \
  -not -path "*/node_modules/*" \
  -not -path "*/.git/*" \
  -not -name "package.json" \
  -not -name "package-lock.json" \
  -not -name "tsconfig.json" \
  -not -name "tsconfig.*.json" \
  \( -name "*.yaml" -o -name "*.yml" -o -name "*.toml" -o -name "*.json" \) \
  2>/dev/null
```

### 2. Classify Each File

Map each discovered file to a known config type:

| Pattern | Type |
|---|---|
| `docker-compose.yml` / `docker-compose.yaml` | Docker Compose |
| `.github/workflows/*.yml` | GitHub Actions |
| `kubernetes/*.yaml` / `k8s/*.yaml` / `*-deployment.yaml` | Kubernetes |
| `pyproject.toml` | Python project (PEP 518/621) |
| `Cargo.toml` | Rust Cargo |
| `.eslintrc.json` / `.eslintrc.yml` / `eslint.config.json` | ESLint |
| `jest.config.json` / `jest.config.js` | Jest |
| `vite.config.ts` / `vite.config.js` | Vite (skip — JS, not config) |
| Other | Unknown / generic |

### 3. Validate by Config Type

#### Docker Compose
Required top-level keys: `version`, `services`
Each service must have either `image` OR `build` (not neither).
Common mistakes:
- Missing `image` and `build` in a service → ERROR
- `ports` value is a number instead of string (e.g. `80` instead of `"80:80"`) → WARN
- `depends_on` referencing a service name that doesn't exist → ERROR
- Tabs used for indentation → ERROR

#### GitHub Actions
Required: `on`, `jobs`
Each job requires `runs-on` and `steps`.
Each step should have either `uses` or `run` (not both, not neither).
Common mistakes:
- Missing `runs-on` → ERROR
- Step has neither `uses` nor `run` → ERROR
- `on` key misspelled as `ON` or `On` (YAML boolean trap) → ERROR (quote it: `"on":`)
- Uses of deprecated actions (e.g. `actions/checkout@v1`) → WARN

#### Kubernetes
Required top-level: `apiVersion`, `kind`, `metadata`, `spec`
`metadata` must have `name`.
Deployment spec must have `selector` and `template`.
Common mistakes:
- Missing `apiVersion` → ERROR
- `kind` typo (e.g. `Deployement`) → ERROR
- `containerPort` is string instead of integer → WARN
- `image` tag is `latest` → WARN (unpinned)

#### pyproject.toml
Must have `[build-system]` with `requires` array and `build-backend` string.
If `[project]` present: `name` and `version` are required.
Common mistakes:
- `requires-python` using wrong format (e.g. `3.9` instead of `">=3.9"`) → WARN
- Missing `build-backend` → ERROR

#### Cargo.toml
Must have `[package]` with `name`, `version`, `edition`.
Common mistakes:
- `edition` not one of `"2015"`, `"2018"`, `"2021"` → ERROR
- `version` not semver → WARN
- Dependency version `"*"` (unpinned) → WARN

#### ESLint
Must be a JSON object.
If `rules` present, each rule value must be `0/1/2` or `"off"/"warn"/"error"` or array with first element being one of those.
Common mistakes:
- `extends` is not a string or array → ERROR
- Unknown rule namespace (e.g. typo in plugin prefix) → WARN

#### Jest
Must be a JSON object.
Common mistakes:
- `testEnvironment` not one of `"node"`, `"jsdom"`, `"jest-environment-*"` → WARN
- `transform` patterns are wrong regex → WARN

#### Generic JSON
- Valid JSON syntax (catch parse errors) → ERROR if malformed
- Duplicate keys → WARN

#### Generic YAML
- Valid YAML syntax → ERROR if malformed
- Tab characters used for indentation → ERROR
- Duplicate keys at same level → WARN

### 4. Format Output

For each issue found, output:

```
[SEVERITY] <file>:<line>
  Type: <error-type>
  Issue: <description>
  Fix: <suggested fix>
```

Severity levels: `ERROR` (breaks functionality), `WARN` (best practice violation), `INFO` (suggestion).

Group by file. At end, print summary:
```
Scanned X files — Y errors, Z warnings
```

If no issues: print `All config files look good.`

### 5. Parsing Notes

- For YAML: read raw text, check for tab characters (`\t` at line start = indentation error)
- For TOML: read raw text, validate `[section]` syntax and `key = value` pairs
- For JSON: use `JSON.parse` equivalent — catch SyntaxError and report line/col
- Line numbers: count newlines up to the error position
