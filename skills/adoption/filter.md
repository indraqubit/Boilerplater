---
name: adoption-filter
version: 1.0
description: |
  Adoption subagent step 3: final pass filter.
  Returns only BLESSED-compliant candidates.
---

# Adoption Filter

**Invoked only when compliance_orchestrator adoption gate fires.**

Receives curated candidates from `curate.md`, applies final BLESSED compliance check.

## Inputs

Received from orchestrator:
```yaml
curated_candidates:
  - original_url: string
    curated_code: string
    transformations_applied: string[]
    compliance_score: float
```

## Behavior

### Step 1: Final BLESSED Check

For each curated candidate, verify:
- **Single Source of Truth**: No state duplicated across objects
- **Explicit Encapsulation**: All fields private, accessed via API only
- **No Hidden Mutation**: `const` methods truly const, no `mutable` abuse
- **No Cyclic Dependencies**: Candidate code does not introduce A→B→A cycles
- **Positive Control Flow**: No early returns

### Step 2: Final JRENG Check

Verify:
- **Bounds Safety**: All array accesses use `.at()` or equivalent
- **RAII/Ownership**: No raw `new`/`delete`
- **No Magic Numbers**: All constants named
- **Correct Constexpr**: `const` used appropriately

### Step 3: Final NAMES Check

Verify:
- **Semantic Naming**: All identifiers meaningful
- **Consistent Style**: Follows project naming conventions

### Step 4: Confidence Scoring

Each passing candidate receives a final confidence score:

```
final_confidence = curation_compliance_score × BLESSED_check × JRENG_check × NAMES_check
```

Where each factor is 0.0 or 1.0 (binary pass/fail on core rules).

### Step 5: Emit Filtered Suggestions

```yaml
filtered_suggestions:
  - original_url: string
    code: string
    confidence: float  # 0.0-1.0
    passes_blessed: bool
    passes_jreng: bool
    passes_names: bool
    blocking_violations: string[]  # if any, candidate excluded
```

Only candidates with `confidence >= 0.8` and no blocking violations are included.

## Output

Filtered suggestions consumed by orchestrator for report synthesis under `Curated Adoptions` section.

## Constraints

- **Confidence threshold: 0.8** — only high-confidence suggestions presented
- **Zero blocking violations** — any BLESSED/JRENG core violation excludes candidate
- Candidates excluded at this stage are logged at advisory level

## Error Handling

If all candidates filtered out:
- Return empty `filtered_suggestions` array
- Report `Adoption pipeline: no candidates passed final filter` at advisory level
- Orchestrator notes "No compliant adoption suggestions available" in report