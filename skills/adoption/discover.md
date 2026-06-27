---
name: adoption-discover
version: 1.0
description: |
  Adoption subagent step 1: discover external boilerplate candidates.
  Invoked only when orchestrator's adoption gate fires.
---

# Adoption Discover

**Invoked only when compliance_orchestrator adoption gate fires.**

## Purpose

Given a violation context, discover external boilerplate candidates from public sources.

## Inputs

Received from orchestrator:
```
{
  rule_id:      string       # The rule that was violated
  file:         string       # File where violation occurred
  line:         number       # Line number of violation
  severity:     critical|major
  violation_type: missing_boilerplate|structural_pattern|repeated_antipattern
  context:      string       # Snippet showing the violation
}
```

## Behavior

### Step 1: Analyze Violation Context

Parse the violation to understand:
- What pattern or boilerplate is missing
- What the expected implementation should provide
- What constraints the project has (JRENG, BLESSED, NAMES rules)

### Step 2: Search External Sources

Search public repositories and registries for matching boilerplate:

```
Sources to search:
- GitHub public repos (via web search or API)
- Community boilerplate repositories
- Framework official examples (JUCE, nlohmann/json, etc.)
```

Search constraints:
- Max 5 candidates per violation
- Candidates must be under permissive license (MIT, BSD, Apache 2.0)
- Exclude candidates with known security issues

### Step 3: Filter Initial Candidates

For each candidate, record:
- `source_url`: repository URL
- `description`: what the boilerplate provides
- `relevance_score`: 0.0–1.0 (how closely it matches the violation)
- `license`: candidate license
- `last_updated`: date of last commit/update

## Output

```yaml
candidates:
  - source_url: string
    description: string
    relevance_score: float
    license: string
    last_updated: string
  ...
```

Returned to orchestrator for passing to `curate.md`.

## Constraints

- **Max 5 candidates per violation** — prevent overwhelming the pipeline
- **Read-only discovery** — no code fetched without user consent
- **No proprietary sources** — open source only

## Error Handling

If search yields no candidates:
- Return empty `candidates` array
- Orchestrator logs `No adoption candidates found for rule_id` at advisory level
- Pipeline continues without adoption suggestions for this violation