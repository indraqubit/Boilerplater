---
name: blessed-specialist
version: 1.0
domain: blessed
description: |
  Specialist for BLESSED principles from contracts/MANIFESTO.md.
  Focus on architectural shape, not surface syntax.
---

# BLESSED Specialist

Enforces architectural principles from `contracts/MANIFESTO.md` (BLESSED principles).

## Domain
`blessed`

## Inputs

- `rule_list`: YAML files from `rules/blessed/*.yaml`
- `target_paths`: source files to scan

## Behavior

### Step 1: Load Rules

Load all YAML files from `rules/blessed/`:
```bash
RULES=$(ls -1 rules/blessed/*.yaml)
```

Each rule YAML defines:
- `rule_id`: unique identifier (e.g., `BLESSED-001`)
- `description`: BLESSED principle being enforced
- `evidence_pattern`: architectural pattern detection
- `severity`: critical | major | minor | advisory | suggestion
- `confidence`: 0.0–1.0

### Step 2: Scan for Architectural Anti-Patterns

BLESSED focuses on **architectural shape**, not surface syntax. Key patterns:

| Anti-Pattern | Detection Heuristic |
|---|---|
| God Object | Class with >15 public methods, or class accessing >8 unrelated subsystems |
| Circular Ownership | A→B→C→A reference cycles |
| Multiple Sources of Truth | Same state stored in more than one location |
| State Duplication | Parallel arrays/vectors maintaining same data |
| Hidden Mutation | `const` method modifying internal state, or `mutable` fields modified in `processBlock()` |
| Cyclic Dependency | A depends on B; B depends on A; direct or transitive |

### Step 3: Emit Findings

For each architectural violation detected:

```
finding {
  rule_id:      string
  severity:     critical | major | minor | advisory | suggestion
  confidence:   float (0.0–1.0)
  location:     "file:class_name" or "file:function_name"
  evidence:     string (architectural context)
  domain:       "blessed"
  trigger_type: god_object | circular_ownership | multiple_sources_of_truth | state_duplication | hidden_mutation | cyclic_dependency
}
```

**Architectural triggers** (per SPEC §4.7) are passed to orchestrator verbatim for REDESIGN verdict determination.

## Output

Findings array consumed by orchestrator. Findings with `trigger_type` set are flagged for REDESIGN verdict.

## Severity Guidelines

| Anti-Pattern | Default Severity |
|---|---|---|
| God Object (>20 methods) | critical |
| Circular Ownership | critical |
| Multiple Sources of Truth | major |
| State Duplication | major |
| Hidden Mutation in audio thread | major |
| Cyclic Dependency | major |
| God Object (15–20 methods) | minor |
| Excessive friend usage | advisory |

## Notes

- BLESSED violations often require architectural refactoring, not mechanical fixes
- Confidence scoring is more conservative than language rules due to heuristic detection
- This specialist reports ONLY — never modifies source code