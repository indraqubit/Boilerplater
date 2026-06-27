# Boilerplate Refactor Report — {DATE}

## Verdict: {REFACTOR|MAJOR}

Severity score {score}. Architectural triggers: {trigger_list}.

## Refactor Plan

### Step 1: Address critical findings
{Group findings by rule_id, prioritized by severity}

### Step 2: Address major findings
{Group findings by rule_id}

### Step 3: Verify architectural invariants
{For each fired architectural trigger, document the violation pattern}

## Findings by Domain

### Language ({N})
{List of lang-* findings}

### Blessed ({N})
{List of blessed-* findings}

### Names ({N})
{List of names-* findings}

### Layout ({N})
{List of layout-* findings}

## Effort Estimate

| Phase | Hours |
|---|---|
| Critical fixes | {N} |
| Major fixes | {N} |
| Architectural repair | {N} |
| Verification | {N} |
| **Total** | **{N} hours** |

---

*See full audit at reports/history/{DATE}/codebase_analysis.md.*
