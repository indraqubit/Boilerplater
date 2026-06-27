# Boilerplater — Specification

**Version:** 1.0
**Date:** 2026-06-28
**Status:** ✅ APPROVED WITH REQUIRED CHANGES (resolutions integrated)
**Repository:** `/Users/indraqadarsih/Downloads/TERRACE/Boilerplater`
**Approver:** ARCHITECT (2026-06-28)

---

## 1. Vision

Boilerplater is a **read-only AI governance layer** for C++/JUCE codebases. It audits code against three contract surfaces — JRENG (mechanical), BLESSED (architectural), NAMES (semantic) — and produces a verifiable `codebase_analysis.md` report. The auditor **never mutates code**. All changes are proposed, reviewed, and applied by humans.

Boilerplater v1 transforms the current single-skill monolith (`jreng_compliance_checker`) into a **modular multi-agent system** with explicit rule IDs, severity tiers, evidence, confidence scoring, dynamic domain discovery, and an extensible capability surface — without premature extraction of concerns that are not yet bounded contexts.

**Vendor-agnostic core.** Boilerplater does not depend on any specific LLM vendor. README may show usage examples for Claude Code, Gemini, ChatGPT, Codex, etc., but the SPEC and core architecture treat all vendors equally (OQ-7 resolved).

---

## 2. Goals (v1)

| ID | Goal | Source |
|---|---|---|
| G1 | Split monolithic `jreng_compliance_checker` into focused subagents per concern | ARCHITECT review §1 |
| G2 | Add stable Rule IDs (with explicit domain prefixes), Severity tiers, Confidence scores, Evidence to every finding | ARCHITECT review §5, §6 |
| G3 | Replace binary verdict with two-dimensional model: severity score + architectural triggers | ARCHITECT decision OQ-5 |
| G4 | Separate concerns by domain folder (rules, skills, templates, reports, capabilities) | ARCHITECT review §3 + OQ addition #4 |
| G5 | Add Rule Engine (rules/ folder, one rule per file, with metadata versioning) | ARCHITECT review §4 + OQ addition #1 |
| G6 | Curated Adoption becomes a conditional subagent, called only when triggered | ARCHITECT decision 2026-06-28 |
| G7 | Maintain extraction path for Curated Adoption as independent repo at v2+ | ARCHITECT decision 2026-06-28 |
| G8 | Dynamic domain discovery — orchestrator scans rules/* and dispatches by discovered domains | OQ addition #5 |
| G9 | Rule metadata supports versioning, tags, and dependencies for graph analysis | OQ additions #1, #2, #3 |
| G10 | Rename `jreng_compliance_checker` → `compliance_orchestrator` with alias for one release | OQ-10 |

## 3. Non-Goals (v1)

| ID | Non-Goal | Rationale |
|---|---|---|
| N1 | Curated Adoption as separate repository | ARCHITECT decision: not yet bounded context |
| N2 | Automatic code mutation | Read-only philosophy (Boilerplater.md) |
| N3 | Git/CLI execution by agents | CAROL.md Git Rules |
| N4 | Performance auditor scope beyond current 3 rules (`noexcept`, pre-increment, range-for) | OQ-9 deferred to v1.1 |
| N5 | Cross-language support beyond C++/JUCE | Out of v1 scope |
| N6 | Real-time audit (incremental file watch) | Out of v1 scope; single-shot audit only |
| N7 | LLM vendor lock-in (treating one vendor as first-class) | OQ-7 resolved: vendor-agnostic core |

---

## 4. Architecture

### 4.1 Directory Tree (v1)

```
Boilerplater/
├── README.md                       # Onboarding (preserved from v0)
├── Boilerplater.md                 # Master philosophy (preserved from v0)
├── CLAUDE.md -> /Users/indraqadarsih/Documents/carol-main/CAROL.md
├── LICENSE                         # Preserved
├── contracts/                      # NEW: Contract abstraction layer (OQ-8)
│   ├── JRENG-CODING-STANDARD.md    # symlink → carol-main
│   ├── MANIFESTO.md                # symlink → carol-main (BLESSED)
│   └── NAMES.md                    # symlink → carol-main
├── rules/                          # NEW: Rule Engine (dynamic discovery)
│   ├── language/
│   │   ├── lang-no-early-return.yaml
│   │   ├── lang-positive-nesting.yaml
│   │   ├── lang-no-anonymous-namespace.yaml
│   │   ├── lang-fail-fast.yaml
│   │   ├── lang-use-at-not-subscript.yaml
│   │   ├── lang-no-magic-numbers.yaml
│   │   └── lang-no-raw-delete.yaml
│   ├── blessed/
│   │   ├── blessed-explicit-encapsulation.yaml
│   │   ├── blessed-single-source-of-truth.yaml
│   │   ├── blessed-stateless.yaml
│   │   ├── blessed-deterministic.yaml
│   │   ├── blessed-lean.yaml
│   │   ├── blessed-bound.yaml
│   │   └── blessed-encapsulation.yaml
│   ├── names/
│   │   ├── names-verb-noun-functions.yaml
│   │   ├── names-no-type-encoding.yaml        # severity: major (override)
│   │   ├── names-cognitive-load.yaml
│   │   ├── names-no-getter-prefix.yaml
│   │   ├── names-no-helper-suffix.yaml        # severity: advisory (default)
│   │   ├── names-domain-terms.yaml
│   │   └── names-no-comments-needed.yaml
│   └── layout/                     # NEW: Layout rules (extracted)
│       ├── layout-includes-order.yaml
│       ├── layout-brace-style.yaml
│       └── layout-line-length.yaml
├── skills/                         # NEW: Agent skill definitions
│   ├── compliance/
│   │   ├── audit.md                # Orchestrator: compliance_orchestrator
│   │   ├── language.md             # Language specialist
│   │   ├── blessed.md              # BLESSED specialist
│   │   ├── names.md                # NAMES specialist
│   │   └── layout.md               # Layout specialist
│   └── adoption/                   # Conditional subagent
│       ├── discover.md
│       ├── curate.md
│       └── filter.md
├── capabilities/                   # NEW: Extension point (OQ addition #4)
│   ├── README.md                   # Capability contract & registration
│   ├── github-search.md            # (planned, not v1-implemented)
│   ├── sarif.md                    # (planned, not v1-implemented)
│   ├── lsp.md                      # (planned, not v1-implemented)
│   ├── semantic-index.md           # (planned, not v1-implemented)
│   └── vector-cache.md             # (planned, not v1-implemented)
├── templates/                      # NEW: Output templates
│   ├── report.md
│   ├── patch.md
│   ├── refactor.md
│   └── redesign.md
├── reports/                        # NEW: Audit output (OQ-4 user-managed)
│   ├── .gitignore                  # Excludes latest.json + history/*
│   ├── latest.json -> history/YYYY-MM-DD/latest.json  # symlink, user updates
│   └── history/
│       └── .gitkeep
├── examples/                       # NEW: Sample audits
│   └── sample-report.md
├── carol/                          # Preserved (symlinks to carol-main)
│   ├── config.yml
│   ├── SPRINT-LOG.md
│   └── [symlinks to contracts]
└── skills/jreng_compliance/        # DEPRECATED in v1 (one-release alias)
    └── SKILL.md -> ../../skills/compliance/audit.md
```

### 4.2 Orchestration Flow (Dynamic Domain Discovery — OQ addition #5)

```
[User invokes compliance audit]
         ↓
[Orchestrator: skills/compliance/audit.md → compliance_orchestrator]
         ↓
[Step 1: DISCOVER DOMAINS]
    Scan rules/* subdirectories
    Build domain index: [language, blessed, names, layout]
    (Future: dsp, audio-thread, etc. — orchestrator unchanged)
         ↓
[Step 2: LOAD RULES]
    For each discovered domain, load all *.yaml rule files
    Build rule registry keyed by rule id
    Validate schema, reject malformed rules
         ↓
[Step 3: PARALLEL DISPATCH]
    For each discovered domain:
        Invoke skills/compliance/<domain>.md specialist
    (Specialists are matched to domains by name, not hardcoded)
         ↓
[Step 4: MERGE FINDINGS]
    Dedupe by (file, line, rule_id) triple
    Keep highest severity + confidence
         ↓
[Step 5: ADOPTION GATE]
    For each finding:
        Internal boilerplate exists in rules/*.yaml? ──── YES ──→ Suggest internal
             │ NO
             ↓
        Trigger conditions:
        - severity == critical OR major
        - violation type matches adoption-eligible patterns
        - user opt-in flag set
             ↓
        [Invoke Adoption subagent?]
        ├── NO  → skip (≈90% of audits)
        └── YES → skills/adoption/{discover → curate → filter}
         ↓
[Step 6: VERDICT COMPUTATION]
    Compute severity_score (sum of severity weights)
    Detect architectural_triggers (god object, circular ownership, etc.)
    Apply two-dimensional verdict model (§4.7)
         ↓
[Step 7: SYNTHESIZE REPORT]
    Render via templates/report.md
    Write to reports/history/YYYY-MM-DD/codebase_analysis.md
    Update reports/latest.json (baseline delta vs previous)
         ↓
[Output]
```

**Critical invariant:** The orchestrator MUST NOT hardcode domain names. It scans `rules/*` at runtime and dispatches specialists by discovered domain. Adding `rules/dsp/` or `rules/audio-thread/` requires zero orchestrator changes (G8).

### 4.3 Rule Engine Schema (Enhanced — OQ additions #1, #2, #3)

Every rule lives in `rules/<domain>/<id>.yaml` with this schema:

```yaml
# === Metadata (OQ addition #1) ===
schema_version: 1                   # YAML schema version (increment on breaking change)
rule_version: 1.2                   # Rule content version (semver: bump on logic change)
status: stable                      # stable | experimental | deprecated

# === Identity ===
id: lang-no-early-return            # MUST include domain prefix (OQ-1)
title: "No Early Returns"
domain: language                    # Must match parent folder

# === Classification ===
severity_default: critical          # critical | major | minor | advisory | suggestion
severity_rationale: |
  Hard rule in JRENG-CODING-STANDARD.md §547 (CRITICAL RULES section).
  Violations break positive-nesting principle and BLESSED Explicit
  Encapsulation.
confidence_floor: high              # high | medium | low
tags:                               # OQ addition #2
  - control-flow
  - readability
  - jreng-critical
depends_on:                         # OQ addition #3
  - blessed-explicit-encapsulation  # Required contract for this rule

# === Description ===
description: |
  Functions must not exit via early `return` statements. Use positive
  nesting only.
rationale: |
  Early returns introduce hidden control flow that violates positive
  nesting and BLESSED Explicit Encapsulation.

# === Detection ===
evidence_pattern: |
  Pattern: `return <expr>;` at function scope, BEFORE the function's
  terminating statement.

# === Examples ===
examples:
  bad:
    - code: |
        int foo(int x) {
          if (x < 0) return -1;   // early return — violation
          return x * 2;
        }
  good:
    - code: |
        int foo(int x) {
          const auto result = (x < 0)
            ? -1
            : x * 2;
          return result;
        }

# === Provenance ===
source_refs:
  - "contracts/JRENG-CODING-STANDARD.md:547"
  - "contracts/MANIFESTO.md (BLESSED Explicit Encapsulation)"
  - "Boilerplater.md:9-12"
author: ARCHITECT
created: 2026-06-28
updated: 2026-06-28
```

### 4.4 Rule ID Scheme (OQ-1 Resolved)

**Format:** `{domain}-{concern}-{specific}` kebab-case, with **explicit domain prefix as part of the ID**.

| Domain | ID prefix | Example |
|---|---|---|
| `language/` | `lang-` | `lang-no-early-return` |
| `blessed/` | `blessed-` | `blessed-explicit-encapsulation` |
| `names/` | `names-` | `names-no-type-encoding` |
| `layout/` | `layout-` | `layout-brace-style` |

**Rationale (ARCHITECT):** Rule IDs appear in reports, JSON exports, CLI output, SARIF, and GitHub annotations. They must stand alone without folder context.

**ID collision check:** All rule IDs are globally unique. Loading two rules with the same ID across domains is a critical error and aborts rule loading.

### 4.5 Severity Tiers (OQ-2 Resolved)

| Tier | Definition | Example |
|---|---|---|
| `critical` | Architectural flaw, hard rule violation, breaks contract | `lang-no-early-return`, `lang-no-raw-delete` |
| `major` | Significant deviation, requires refactor; affects maintainability | `names-no-type-encoding`, `lang-use-at-not-subscript` |
| `minor` | Style/naming convention deviation | `layout-brace-style`, `layout-line-length` |
| `advisory` | Informational, suggest improvement. **Default for NAMES rules** | `names-cognitive-load`, `names-no-helper-suffix` |
| `suggestion` | Nice-to-have, never blocks | Naming alternative proposal |

**Severity defaults per domain:**

| Domain | Default | Override examples |
|---|---|---|
| `language/` | `critical` | `lang-use-at-not-subscript` → `major` |
| `blessed/` | `critical` | `blessed-lean` → `major` |
| `names/` | **`advisory`** | `names-no-type-encoding` → `major` |
| `layout/` | `minor` | (none in v1) |

Severity is set per-rule in `severity_default`. Project config may override.

### 4.6 Confidence Tiers

Per Librarian research (Semgrep-derived):

| Tier | When |
|---|---|
| `high` | Direct pattern match with full context (e.g., AST-level) |
| `medium` | Pattern-only match, no semantic context |
| `low` | Regex heuristic, name-based inference |

Confidence floor per rule in YAML. Findings below floor are **suppressed** (OQ-3 resolved).

### 4.7 Verdict Scale (Two-Dimensional — OQ-5 Resolved)

Replaces current binary (REDESIGN / 5 MINS CHECKLIST DIFF) and previous count-based proposal.

**Two dimensions:**

**A. Severity Score** (weighted sum of findings):
```
severity_score = (count_critical × 10) + (count_major × 5) + (count_minor × 2) + (count_advisory × 1) + (count_suggestion × 0.5)
```

**B. Architectural Triggers** (boolean, OR-combined):
- `god_object` — single class with excessive responsibilities
- `circular_ownership` — cycles in ownership graph
- `multiple_sources_of_truth` — same state owned by ≥2 components
- `state_duplication` — mirrored state across boundaries
- `hidden_mutation` — non-obvious side effects
- `cyclic_dependency` — module-level dependency cycles

**Verdict mapping:**

| Verdict | Trigger |
|---|---|
| `PASS` | severity_score = 0 AND no architectural_triggers |
| `PATCH` | severity_score ≤ 10 AND no architectural_triggers |
| `REFACTOR` | 10 < severity_score ≤ 30 AND no architectural_triggers |
| `MAJOR` | severity_score > 30 AND no architectural_triggers |
| `REDESIGN` | **any architectural_trigger fires** OR severity_score > 50 |

**Why two-dimensional:** A single God Object is REDESIGN even with 0 rule violations. Conversely, 30 minor findings may not warrant REDESIGN if architecture is clean. Severity alone is insufficient signal; architectural shape matters more than count.

---

## 5. Audit Report Format

Output: `reports/history/YYYY-MM-DD/codebase_analysis.md`
Baseline pointer: `reports/latest.json` (user-managed symlink, OQ-4 resolved)

```markdown
# Boilerplate Analysis — YYYY-MM-DD

## Summary

| Metric | Value |
|---|---|
| Files scanned | N |
| Total findings | N |
| Severity score | N |
| Architectural triggers | [list of fired triggers] |
| Verdict | PASS \| PATCH \| REFACTOR \| MAJOR \| REDESIGN |
| Confidence coverage | high: N / medium: N / low: N |

## Findings by Severity

| Tier | Count |
|---|---|
| critical | N |
| major | N |
| minor | N |
| advisory | N |
| suggestion | N |

## Findings (Detail)

### [rule-id] — [title]

- **Severity:** [tier]
- **Confidence:** [tier]
- **Location:** `path/to/file.cpp:line`
- **Tags:** [from rule's tags field]
- **Depends on:** [from rule's depends_on field]
- **Evidence:** [code snippet, ≤3 lines]
- **Reason:** [why this violates]
- **Expected:** [what compliant code looks like]
- **Suggested fix:** [concrete diff]
- **Source refs:** [doc citations]

[repeat per finding]

## Adoption Suggestions (if triggered)

[Adoption subagent output, only if trigger conditions met]

## Baseline Delta

| Metric | Last audit (latest.json) | Today | Δ |
|---|---|---|---|
| Severity score | N | N | ±N |
| Critical | N | N | ±N |
| Major | N | N | ±N |
| Architectural triggers | [list] | [list] | ± |

## ROI Estimate

[Time saved / cost per verdict tier, preserved from v0 SKILL.md:47-52]
```

### 5.1 Report Storage (OQ-4 Resolved)

```
reports/
├── latest.json -> history/2026-06-28/latest.json   # user-managed symlink
└── history/
    └── 2026-06-28/
        ├── codebase_analysis.md
        └── latest.json
    └── 2026-06-29/
        └── ...
```

**Retention policy: user-managed.** Boilerplater NEVER auto-deletes audit history. The user controls:
- When to advance `latest.json`
- When to archive old history
- When to delete (never automatic)

This avoids hidden data loss and makes baseline tracking fully transparent.

---

## 6. Constraints

### 6.1 Read-Only Invariant

The auditor MUST NEVER mutate project source code. All outputs go to `reports/`. Any code suggestion is presented as a diff snippet in the report, never applied to disk.

Source: `Boilerplater.md` philosophy + ARCHITECT review §3.

### 6.2 Contract Surface (OQ-8 Resolved)

The auditor enforces three contract docs via a **`contracts/` abstraction layer**:

```
contracts/
├── JRENG-CODING-STANDARD.md -> /Users/indraqadarsih/Documents/carol-main/JRENG-CODING-STANDARD.md
├── MANIFESTO.md             -> /Users/indraqadarsih/Documents/carol-main/MANIFESTO.md
└── NAMES.md                 -> /Users/indraqadarsih/Documents/carol-main/NAMES.md
```

**Invariant:** Boilerplater core code references ONLY `contracts/` paths. The implementation detail (symlink, copy, submodule) is hidden. Swapping contracts/ from symlink to vendored docs requires zero changes to rule files or skills.

### 6.3 Vendor-Agnostic Core (OQ-7 Resolved)

Boilerplater does not depend on any specific LLM vendor. Core assumptions:
- Skills are markdown (portable across Claude Code, Gemini, ChatGPT, Codex, Cursor, Roo, Cline)
- Rule files are pure YAML (no vendor extensions)
- Audit output is plain Markdown + JSON (no proprietary formats)

README may show usage examples for multiple vendors, but **no vendor is "first-class" in the core**.

### 6.4 Adoption Extraction Path (G7)

The contract for `skills/adoption/*` MUST be designed so that:
- Discovery, curation, and filtering are independently invokable
- Each skill has a clearly-defined input/output interface (input: violation context; output: filtered suggestion)
- No skill directly imports state from Boilerplater core; all communication via the orchestrator's context dict

This allows future extraction (v2+) into `Boilerplater-Adoption/` repo without changing the caller interface.

### 6.5 Backward Compatibility & Rename (OQ-10 Resolved)

**Rename:** `jreng_compliance_checker` → `compliance_orchestrator`

**Migration path:**
1. v1.0: New name is primary. Old name retained as alias (`skills/jreng_compliance/SKILL.md` symlinks to `skills/compliance/audit.md`).
2. v1.1: Old name marked deprecated in README.
3. v2.0: Old name removed.

README.md MUST include a deprecation notice section explaining the rename and alias window.

### 6.6 CAROL Compliance

- All agents built per CAROL.md role separation
- COUNSELOR writes `SPEC.md`, `PLAN.md`, `ARCHITECTURE.md` (not delegated)
- Engineer implements; Auditor validates; SURGEON executes surgical changes
- Git operations: ARCHITECT only (CAROL.md Git Rules)

### 6.7 Capabilities Extension Point (OQ addition #4)

`capabilities/` is a reserved directory for future optional features. v1 includes only `capabilities/README.md` documenting the capability contract. Capabilities may include:

- `github-search` — search external sources
- `sarif` — SARIF output format
- `lsp` — Language Server Protocol integration
- `semantic-index` — semantic code indexing
- `vector-cache` — vector embedding cache

Capabilities are NOT loaded into v1 audit. They exist as the architectural extension surface.

---

## 7. Edge Cases

| Case | Behavior |
|---|---|
| Rule file has malformed YAML | Log error, skip rule, continue audit. Report rule-loading failures in summary. |
| Rule `schema_version` mismatch | Warn but load (forward-compat). Critical mismatch aborts. |
| Rule `status: deprecated` | Load but emit advisory note in report; do not flag as violation. |
| No rules in a domain folder | Skip that domain in dispatch. No findings. |
| Adoption trigger fires but external search returns no results | Return empty adoption section with note "No external boilerplate matched". |
| Same violation found by multiple specialists | Dedupe by `(file, line, rule_id)` triple. Keep highest severity and confidence. |
| Rule ID collision across domains | Abort rule loading. Critical error. Fix rule file naming. |
| Audit target is empty directory | Return PASS verdict with note "No source files to scan". |
| Confidence floor for rule is `low` and finding matches | Suppress (OQ-3). |
| `depends_on` rule not loaded | Warn at load; rule still active. Dependency is informational, not gating. |
| Dynamic domain discovered without matching specialist | Emit warning; rules in that domain still loaded and reported. Specialist dispatch skipped. |
| `latest.json` symlink broken or missing | Report baseline delta as "N/A (no baseline)". No error. |
| Performance auditor rule missing for new pattern | Out of scope v1 (N4). Document gap in report. |

---

## 8. Acceptance Criteria

Each criterion is verifiable by a single machine action.

### 8.1 Repository Structure

- [ ] `rules/` directory exists with `language/`, `blessed/`, `names/`, `layout/` subdirs
- [ ] `skills/compliance/` contains `audit.md`, `language.md`, `blessed.md`, `names.md`, `layout.md`
- [ ] `skills/adoption/` contains `discover.md`, `curate.md`, `filter.md`
- [ ] `capabilities/` exists with `README.md` (capability contract)
- [ ] `contracts/` exists with symlinks to JRENG, MANIFESTO, NAMES
- [ ] `templates/` contains `report.md`, `patch.md`, `refactor.md`, `redesign.md`
- [ ] `reports/` exists with `.gitignore`, `history/`, and `latest.json` placeholder
- [ ] `examples/` exists with at least one sample report
- [ ] `skills/jreng_compliance/SKILL.md` exists as symlink to `skills/compliance/audit.md`
- [ ] All rule files use explicit domain prefixes in IDs

### 8.2 Rule Engine

- [ ] At least one rule file exists per domain (`language`, `blessed`, `names`, `layout`)
- [ ] Each rule YAML validates against the schema in §4.3 (schema_version, rule_version, status, id, severity_default, confidence_floor, source_refs)
- [ ] Total rule count ≥ 24 (7 lang + 7 blessed + 7 names + 3 layout)
- [ ] Every rule has a unique `id` with explicit domain prefix
- [ ] Every rule has `tags` array (may be empty for v1 minimum)
- [ ] Every rule has `depends_on` array (may be empty for v1 minimum)
- [ ] `names-no-type-encoding` has `severity_default: major`
- [ ] `names-no-helper-suffix` has `severity_default: advisory`

### 8.3 Orchestrator Behavior (G8)

- [ ] Audit invocation produces a `reports/history/YYYY-MM-DD/codebase_analysis.md` file
- [ ] Orchestrator scans `rules/*` at runtime to discover domains (verified by grep: no hardcoded domain list in `skills/compliance/audit.md`)
- [ ] Report includes summary table, severity histogram, severity score, architectural triggers list, findings detail, baseline delta, ROI
- [ ] Verdict is one of: `PASS`, `PATCH`, `REFACTOR`, `MAJOR`, `REDESIGN`
- [ ] Verdict computation uses two-dimensional model (severity_score + architectural_triggers)
- [ ] Adoption subagent is NOT invoked when no critical/major findings exist (verified by orchestrator output)

### 8.4 Backward Compatibility (OQ-10)

- [ ] `skills/jreng_compliance/SKILL.md` symlink resolves to `skills/compliance/audit.md`
- [ ] README.md includes deprecation notice for `jreng_compliance_checker` pointing to `compliance_orchestrator`
- [ ] v0.x audit format option documented (legacy flag) in README

### 8.5 Vendor-Agnostic (OQ-7)

- [ ] No rule file references a specific LLM vendor
- [ ] No skill file is gated behind vendor-specific syntax
- [ ] Core Markdown + YAML only (no proprietary extensions)

### 8.6 Contract Abstraction (OQ-8)

- [ ] `contracts/` directory exists with three symlinks
- [ ] Rule `source_refs` paths begin with `contracts/` (not raw carol-main paths)
- [ ] No skill or rule file references `carol-main` directly

### 8.7 Capabilities Extension (OQ addition #4)

- [ ] `capabilities/README.md` exists and defines the capability registration contract
- [ ] No v1 audit flow loads or requires any capability
- [ ] Adding a new capability file does NOT require changes to orchestrator

### 8.8 Spec Traceability

- [ ] Every ARCHITECT review item maps to at least one SPEC goal (G1–G10)
- [ ] Every Open Question from BRAINSTORMER is resolved in this SPEC (table below)

---

## 9. Resolved Decisions (OQ Status)

| OQ | Decision | Section |
|---|---|---|
| OQ-1 | Explicit domain prefixes (`lang-`, `blessed-`, `names-`, `layout-`) | §4.4 |
| OQ-2 | NAMES default = `advisory`; per-rule override permitted | §4.5 |
| OQ-3 | Suppress findings below confidence floor | §4.6 |
| OQ-4 | User-managed retention; baseline as `latest.json` symlink | §5.1 |
| OQ-5 | Two-dimensional verdict: severity score + architectural triggers | §4.7 |
| OQ-6 | One rule per YAML file | §4.3 |
| OQ-7 | Vendor-agnostic core; no first-class vendor | §6.3 |
| OQ-8 | Symlink via `contracts/` abstraction | §6.2 |
| OQ-9 | Performance scope deferred to v1.1 | N4 |
| OQ-10 | Rename `jreng_compliance_checker` → `compliance_orchestrator` with alias | §6.5 |

---

## 10. Out of Scope (v1)

- Performance auditor with audio-thread rules (deferred to v1.1)
- Curated Adoption as separate repository (deferred to v2+)
- Real-time/incremental audit
- Non-C++ language support
- Web UI for report visualization
- CI/CD pipeline integration
- Capabilities (SARIF, LSP, semantic-index, etc.) — directory exists but not loaded
- Auto-deletion of report history

---

## 11. References

### Source Documents (Current Repo)
- `Boilerplater.md` — philosophy
- `README.md` — onboarding
- `skills/jreng_compliance/SKILL.md` — v0 audit skill (deprecated)

### Contract Docs (via contracts/ abstraction)
- `contracts/JRENG-CODING-STANDARD.md` (587 lines, symlinked)
- `contracts/MANIFESTO.md` (305 lines, symlinked)
- `contracts/NAMES.md` (326 lines, symlinked)

### External Patterns
- Semgrep YAML rule schema (rule metadata, severity, confidence tiers)
- LangGraph supervisor orchestration pattern
- ESLint rule structure (rule ID conventions)

### ARCHITECT Inputs
- Architectural review dated 2026-06-28 (8.8/10 scoring)
- Scope decision: Adoption as subagent with v2+ extraction path
- APPROVED WITH REQUIRED CHANGES dated 2026-06-28 (10 OQ decisions + 5 additions)

---

## 12. Approval

**Status:** ✅ APPROVED WITH REQUIRED CHANGES (resolutions integrated 2026-06-28)

**Approver:** ARCHITECT
**Date:** 2026-06-28

This SPEC is locked. PLAN.md may now be written.