---
name: names-specialist
version: 1.0
domain: names
description: |
  Specialist for semantic naming rules from contracts/NAMES.md.
  Default severity treatment: advisory (per OQ-2).
---

# Names Specialist

Enforces semantic naming rules from `contracts/NAMES.md`.

## Domain
`names`

## Inputs

- `rule_list`: YAML files from `rules/names/*.yaml`
- `target_paths`: source files to scan

## Behavior

### Step 1: Load Rules

Load all YAML files from `rules/names/`:
```bash
RULES=$(ls -1 rules/names/*.yaml)
```

Each rule YAML defines:
- `rule_id`: unique identifier (e.g., `NAMES-001`)
- `description`: naming rule being enforced
- `violation_pattern`: regex for identifier detection
- `severity`: override from default advisory
- `confidence`: 0.0–1.0

### Step 2: Scan Identifiers

Scan function names, class names, variable names, namespace names for violations:

| Violation Type | Detection |
|---|---|
| Hungarian notation | Identifiers starting with `b`, `n`, `i`, `str`, `p` + type hints |
| Type encoding | `FilmstripLookAndFeel`, `converterKnob`, `xmlPtr` |
| No verb for functions | Functions named as nouns (`Calculate`, `Handler` without action) |
| No noun for data | Variables named as verbs (`processing`, `managing`) |
| Abbreviations | `ptr`, `ctx`, `mgr`, `util` instead of full words |
| Underscore abuse | Leading `_` or multiple `__` in identifiers |

### Step 3: Emit Findings

```
finding {
  rule_id:      string
  severity:     critical | major | minor | advisory | suggestion
  confidence:   float (0.0–1.0)
  location:     "file:line:identifier"
  evidence:     string (identifier name + context)
  domain:       "names"
  suggested_fix: string (recommended rename, if applicable)
}
```

## Severity Default: Advisory

Per SPEC §4.5 and OQ-2, naming violations default to `advisory` severity.

**Severity Overrides:**

| Violation Type | Override Severity |
|---|---|
| Breaks build (reserved name collision) | critical |
| Semantic overload (same name for different concepts) | major |
| Systematic pattern violation (>5 instances) | minor |
| Single instance, minor | advisory |
| Style preference | suggestion |

## Output

Findings array consumed by orchestrator. Low severity means naming findings do not heavily weight the severity score.

## Notes

- Naming violations are the lowest-weighted domain in verdict computation
- Findings include `suggested_fix` field to aid remediation
- This specialist reports ONLY — never modifies source code
- Confidence below 0.6 is common due to context-dependent naming judgment