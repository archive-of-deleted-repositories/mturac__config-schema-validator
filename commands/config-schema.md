# /config-schema

**Trigger**: `/config-schema [file]`

**Purpose**: Infer a JSON Schema from an existing config file and save it to `.config-schemas/`.

**Arguments**:
- `[file]` (required): path to the config file to analyze (YAML, TOML, or JSON)

**Behavior**:
- Invokes the `skills/config-schema` skill
- Reads and parses the specified file
- Infers field types, required vs optional, and patterns
- Saves JSON Schema (Draft 7) to `.config-schemas/<filename>.schema.json`
- Future `/config-validate` runs will also check against this custom schema

**Example**:
```
/config-schema myapp.config.json

Inferring schema from myapp.config.json...

{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "myapp.config.json configuration",
  "type": "object",
  "required": ["host", "port", "database"],
  "properties": {
    "host": { "type": "string" },
    "port": { "type": "integer" },
    "database": {
      "type": "object",
      "required": ["url", "pool_size"],
      "properties": {
        "url": { "type": "string", "description": "File path" },
        "pool_size": { "type": "integer" }
      }
    },
    "debug": { "type": "boolean" }
  },
  "additionalProperties": false
}

Schema saved: .config-schemas/myapp.config.json.schema.json
Fields: 3 required, 1 optional
```
