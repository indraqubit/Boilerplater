---
name: compliance_orchestrator
version: 1.0
status: stable
description: |
  Boilerplater compliance auditor. Performs dynamic domain discovery over
  rules/, dispatches specialists in parallel, computes two-dimensional verdict,
  and triggers adoption subagent only on critical/major findings.
deprecated_alias: jreng_compliance_checker
---

# Compliance Orchestrator

Orchestrates dynamic domain discovery and parallel specialist dispatch for codebase compliance auditing.

## 1. Identity & Vendor Neutrality

This skill is **vendor-agnostic**. It operates identically across:
- Claude Code
- Gemini
- ChatGPT
- Codex
- Cursor
- Roo
- Cline
- Any LLM agent supporting markdown skill dispatch

No vendor-specific syntax, tool APIs, or model features are required. All logic expressed in markdown with bash pseudocode for discovery. This skill emits findings consumed by the invoking agent for report synthesis.

## 2. Dynamic Domain Discovery

**CRITICAL: This orchestrator does NOT hardcode the domain list.**

Domains are discovered at runtime by scanning `rules/*/` subdirectories. The set of discovered domains is dynamic:

```
rules/
  language/    # C++ mechanical rules
  blessed/     # BLESSED principle checks
  names/       # Semantic naming rules
  layout/      # Include order, brace style, line length
  ...          # future domains added without editing this file
```

### Discovery Protocol

**Step 1 — Discover domains at runtime:**

```bash
# Discover all domain directories under rules/
DOMAINS=$(ls -1 rules/ | grep -v '^\\.')
# DOMAINS is dynamic — no hardcoded list in this file
```

**Step 2 — Load rules per discovered domain:**

```bash
for DOMAIN in $DOMAINS; do
  RULES_${DOMAIN}=$(ls -1 rules/${DOMAIN}/*.yaml 2>/dev/null || true)
done
```

**Step 3 — Dispatch specialist by discovered domain:**

```bash
for DOMAIN in $DOMAINS; do
  SPECIALIST="skills/compliance/${DOMAIN}.md"
  if [[ -f "$SPECIALIST" ]]; then
    # Dispatch specialist with RULES_${DOMAIN}
  else
    # Emit warning: no specialist for discovered domain, continue
    echo "WARNING: no specialist for domain '$DOMAIN', loading rules directly"
  fi
done
```

**Behavior if no domains discovered:**

If `rules/` is empty or contains no subdirectories, emit warning and return empty findings array.

## 3. Parallel Specialist Dispatch

After domain discovery, dispatch all available specialists in parallel:

| Specialist | Domain | Source |
|---|---|---|
| `skills/compliance/language.md` | `language` | `rules/language/*.yaml` |
| `skills/compliance/blessed.md` | `blessed` | `rules/blessed/*.yaml` |
| `skills/compliance/names.md` | `names` | `rules/names/*.yaml` |
| `skills/compliance/layout.md` | `layout` | `rules/layout/*.yaml` |

Each specialist receives:
- `rule_list`: YAML files for its domain
- `target_paths`: source files to scan

Each specialist returns:
- `findings[]`: array of finding objects consumed by orchestrator

Findings from all specialists are merged into a single context dictionary for verdict computation.

## 4. Adoption Gate (Conditional)

**The adoption subagent MUST only be invoked when ALL conditions hold:**

1. `severity` is `critical` OR `major`
2. `violation_type` matches adoption-eligible patterns:
   - `missing_boilerplate`
   - `structural_pattern`
   - `repeated_antipattern`
3. `user_opt_in` flag is set (default: `true` for v1)

**Approximately 90% of audits will NOT trigger adoption.** This is by design (per SPEC §6.4).

If all conditions hold, invoke adoption pipeline in sequence:

```
discover.md → curate.md → filter.md
```

### Adoption Pipeline Inputs/Outputs

```
discover.md
  IN:  violation_context (rule_id, file, line, severity)
  OUT: candidate_list (max 5 per violation)

curate.md
  IN:  candidate_list + project_context
  OUT: curated_candidates (JRENG/BLESSED/NAMES filtered)

filter.md
  IN:  curated_candidates
  OUT: filtered_suggestions (BLESSED-compliant, confidence-scored)
```

Filtered suggestions are inserted into the report under `Curated Adoptions` section.

## 5. Verdict Computation (Two-Dimensional)

### Severity Score

```
severity_score = (count_critical × 10)
               + (count_major    × 5)
               + (count_minor    × 2)
               + (count_advisory × 1)
               + (count_suggestion × 0.5)
```

### Architectural Triggers (any → REDESIGN)

- `god_object`
- `circular_ownership`
- `multiple_sources_of_truth`
- `state_duplication`
- `hidden_mutation`
- `cyclic_dependency`

### Verdict Mapping

| Verdict | Trigger Condition |
|---|---|
| **PASS** | `score = 0` AND no architectural triggers |
| **PATCH** | `score ≤ 10` AND no triggers |
| **REFACTOR** | `10 < score ≤ 30` AND no triggers |
| **MAJOR** | `score > 30` AND no triggers |
| **REDESIGN** | any architectural trigger OR `score > 50` |

## 6. Report Generation

Render report via `templates/report.md`.

Output path pattern:
```
reports/history/YYYY-MM-DD/codebase_analysis.md
```

Update `reports/latest.json` baseline pointer to most recent report.

### Report Sections

1. **Executive Summary** — verdict, score, finding counts by severity
2. **JRENG Violations** — from language specialist
3. **BLESSED Violations** — from blessed specialist
4. **Naming Violations** — from names specialist
5. **Layout Violations** — from layout specialist
6. **Curated Adoptions** — from adoption pipeline (if triggered)
7. **ROI Estimate** — see §9
8. **Action Statement** — one of: REDESIGN | REFACTOR | PATCH | 5 MINS CHECKLIST DIFF

## 7. Read-Only Invariant

**This orchestrator MUST NEVER mutate project source code.**

- Outputs go to `reports/` only
- Code suggestions are diff snippets embedded in the report
- No modifications applied to source tree
- Adoption pipeline suggestions are recommendations only

## 8. Source Contracts

All rule references go through `contracts/`:

| Contract | Purpose |
|---|---|
| `contracts/JRENG-CODING-STANDARD.md` | C++ mechanical rules |
| `contracts/MANIFESTO.md` | BLESSED principles |
| `contracts/NAMES.md` | Semantic naming rules |

**Never reference `carol-main` directly.** Use the `contracts/` symlinks.

## 9. ROI Estimate

| Verdict | Time to Fix (est.) | Cognitive Load Reduction | Bug Prevention |
|---|---|---|---|
| PASS | 0 min | None needed | Maintained |
| PATCH | 5–15 min | Minor | Low |
| REFACTOR | 30–120 min | Moderate | Medium |
| MAJOR | 2–8 hrs | Significant | High |
| REDESIGN | 1–3 days | Architectural | Critical |

**ROI Factors:**
- New developer onboarding time reduction (semantic naming)
- Maintenance hours saved (explicit boundaries, no hidden state)
- Bug prevention value (bounds checking, RAII, positive checks)
- Long-term velocity gain from standardized patterns