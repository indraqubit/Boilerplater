---
name: adoption-curate
version: 1.0
description: |
  Adoption subagent step 2: rewrite candidates to match project constraints.
---

# Adoption Curate

**Invoked only when compliance_orchestrator adoption gate fires.**

Receives candidates from `discover.md`, rewrites them to comply with project rules.

## Inputs

Received from orchestrator:
```yaml
candidates:
  - source_url: string
    description: string
    relevance_score: float
    license: string
    last_updated: string
project_context:
  rules: # Merged rules from all domains for constraint checking
  language_rules: [...]
  blessed_rules: [...]
  names_rules: [...]
```

## Behavior

### Step 1: Fetch and Parse Candidates

For each candidate:
- Fetch source from `source_url`
- Parse to identify the core pattern/boilerplate
- Extract the transferable implementation

### Step 2: Apply JRENG Filter

Rewrite code to satisfy `contracts/JRENG-CODING-STANDARD.md`:
- Replace raw array `[]` with `.at()` or safe alternative
- Remove early returns — convert to positive nested checks
- Eliminate magic numbers — define as named constants
- Remove raw `new`/`delete` — use smart pointers or value semantics
- Fix any mechanical violations

### Step 3: Apply BLESSED Filter

Rewrite code to satisfy `contracts/MANIFESTO.md`:
- Ensure single source of truth
- Remove hidden mutation
- Enforce explicit encapsulation (no direct field access from outside)
- Break any circular dependencies

### Step 4: Apply NAMES Filter

Rewrite code to satisfy `contracts/NAMES.md`:
- Rename identifiers to semantic names
- Remove type encoding from names
- Use verbs for functions, nouns for data
- Expand abbreviations

### Step 5: Emit Curated Candidates

```yaml
curated_candidates:
  - original_url: string
    curated_code: |
      # Fully JRENG/BLESSED/NAMES compliant code
    transformations_applied:
      - "Removed early return at line X"
      - "Replaced [] with .at()"
      - "Renamed xmlPtr → xmlParser"
    compliance_score: float  # 0.0-1.0 after filtering
```

## Output

Curated candidates array passed to `filter.md` for final compliance check.

## Constraints

- If candidate cannot be made compliant, exclude from output
- Track all transformations applied for transparency
- Preserve license attribution in curated code comments

## Error Handling

If curation fails for all candidates:
- Return empty `curated_candidates` array
- Include `curation_failure_reason` per candidate