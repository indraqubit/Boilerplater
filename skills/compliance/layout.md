---
name: layout-specialist
version: 1.0
domain: layout
description: |
  Specialist for layout rules. Lowest priority dispatch.
---

# Layout Specialist

Enforces mechanical layout rules: include order, brace style, line length.

## Domain
`layout`

## Inputs

- `rule_list`: YAML files from `rules/layout/*.yaml`
- `target_paths`: source files to scan

## Behavior

### Step 1: Load Rules

Load all YAML files from `rules/layout/`:
```bash
RULES=$(ls -1 rules/layout/*.yaml)
```

Each rule YAML defines:
- `rule_id`: unique identifier (e.g., `LAYOUT-001`)
- `description`: layout rule being enforced
- `evidence_pattern`: detection pattern
- `severity`: minor | advisory | suggestion
- `confidence`: 0.0–1.0

### Step 2: Scan for Layout Violations

| Pattern | Detection |
|---|---|
| Include order | `#include` not alphabetical or not grouped (system → third-party → local) |
| Brace style | `{` not on same line or alone on next line (project convention) |
| Line length | Lines exceeding 120 characters |
| Trailing whitespace | Lines ending with space/tab |
| Tabs vs spaces | Inconsistent indentation |
| Missing newline EOF | File does not end with newline |

### Step 3: Emit Findings

```
finding {
  rule_id:      string
  severity:     minor | advisory | suggestion
  confidence:   float (0.0–1.0)
  location:     "file:line"
  evidence:     string (snippet showing violation)
  domain:       "layout"
}
```

## Severity: Lowest Priority

Layout violations carry the lowest severity weight:
- `minor` → 2 points
- `advisory` → 1 point
- `suggestion` → 0.5 points

This is the last specialist dispatched (lowest priority).

## Output

Findings array consumed by orchestrator.

## Notes

- Layout is lowest priority domain — findings rarely affect verdict except at margin
- This specialist reports ONLY — never modifies source code
- Most layout findings can be auto-fixed with standard formatters