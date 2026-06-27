# Boilerplater — Architecture

**Version:** 1.0
**Date:** 2026-06-28

This document describes the current architecture as implemented. Source of truth: the directory tree and files at this repository. Where this document and the implementation diverge, the implementation wins — update this document.

---

## System Overview

Boilerplater is a **read-only AI governance layer** that audits C++/JUCE codebases against three contract surfaces: JRENG (mechanical), BLESSED (architectural), NAMES (semantic). It produces a verifiable Markdown report. The auditor **never mutates project code**.

v1 introduces a modular multi-agent architecture with explicit rule IDs, severity tiers, evidence, confidence scoring, dynamic domain discovery, and an extensible capability surface.

## Components

### 1. Contracts Layer

`contracts/` is the indirection layer over CAROL framework docs. Six symlinks (3 legacy C++ + 3 v1.1-preview JS/TS):

**v1 stable (C++ rules):**
- `contracts/JRENG-CODING-STANDARD.md` → `~/Documents/carol-main/JRENG-CODING-STANDARD.md`
- `contracts/MANIFESTO.md` → `~/Documents/carol-main/MANIFESTO.md`
- `contracts/NAMES.md` → `~/Documents/carol-main/NAMES.md`

**v1.1 preview (JS/TS rules — inert, staged for activation):**
- `contracts/CODING-CONTRACT.md` → Vortex Realm JWT-2 universal JS/TS rules (27 rules: U1–U6 universal, LAW 1–4 scripts, F1–F7 functions, FL1–FL10 frontend)
- `contracts/FRONTEND_LAW_VAULT.md` → Vortex Realm JWT-2 React+CSS laws (11 laws: LAW 0–10)
- `contracts/BASELINE_LAWS_INDEX.md` → Vortex Realm JWT-2 baseline law index (6 law systems)

These v1.1 contracts are inert — no v1 rule or skill references them. They exist as the contract abstraction layer being staged forward per SPEC §6.2. **Activation trigger:** When `rules/js/` is added (v1.1), these contracts will be referenced by `source_refs` per the established schema.

The orchestrator and rule files reference ONLY `contracts/` paths. The implementation detail (symlink, copy, submodule) is hidden from rule logic.

### 2. Rule Engine

`rules/<domain>/<id>.yaml` — 24 rule files across 4 domains:

| Domain | Count | Default Severity |
|---|---|---|
| `language/` | 7 | critical |
| `blessed/` | 7 | critical |
| `names/` | 7 | advisory (with per-rule overrides) |
| `layout/` | 3 | minor |

Each rule has:
- `schema_version: 1`, `rule_version: 1.0`, `status: stable`
- `id` with explicit domain prefix (`lang-`, `blessed-`, `names-`, `layout-`)
- `severity_default`, `confidence_floor`, `tags`, `depends_on`
- `source_refs` paths starting with `contracts/`

### 3. Skills Layer (Orchestrator + Specialists + Adoption)

#### Orchestrator: `skills/compliance/audit.md`
**Name:** `compliance_orchestrator` (renamed from `jreng_compliance_checker`)
**Behavior:** Performs dynamic domain discovery over `rules/*` at runtime. No hardcoded domain list. The orchestrator:
1. Discovers domains by scanning `rules/*/`
2. Loads rules per domain
3. Dispatches specialists in parallel
4. Merges findings
5. Applies adoption gate (only for critical/major with opt-in)
6. Computes two-dimensional verdict (severity score + architectural triggers)
7. Synthesizes report via `templates/report.md`

**Backward compatibility:** `skills/jreng_compliance/SKILL.md` is a symlink alias to the canonical orchestrator. v0 file preserved as `SKILL.v0-deprecated.md`. Alias removed in v2.0.

#### Specialists: `skills/compliance/{language,blessed,names,layout}.md`
Each specialist receives its domain's rule list and emits findings.

#### Adoption Subagent: `skills/adoption/{discover,curate,filter}.md`
Conditional, invoked only when adoption gate fires (~10% of audits). Each step has explicit input/output contracts to enable future extraction into a separate repo.

### 4. Templates

`templates/{report,patch,refactor,redesign}.md` — output skeletons. Each verdict level has a focused template; `report.md` is the comprehensive base.

### 5. Capabilities (Extension Point)

`capabilities/` — reserved for future optional features. `capabilities/README.md` defines the capability contract. No v1 capability is loaded into the audit pipeline. Planned: `github-search`, `sarif`, `lsp`, `semantic-index`, `vector-cache`.

### 6. Reports

`reports/history/YYYY-MM-DD/codebase_analysis.md` — audit output directory.
`reports/latest.json` — user-managed baseline pointer (symlink or copy).

Retention is **user-managed**. Boilerplater never auto-deletes audit history.

### 7. Examples

`examples/sample-report.md` — synthetic filled example demonstrating all sections (severity scoring, architectural triggers, baseline delta, ROI estimate). Uses all 24 real rule IDs.

## Data Flow

```
User invokes compliance audit
   ↓
Orchestrator (compliance_orchestrator)
   ↓
[1] DISCOVER DOMAINS — scan rules/* (dynamic)
   ↓
[2] LOAD RULES — validate schema, build rule registry
   ↓
[3] PARALLEL DISPATCH — invoke specialists by discovered domain
   ↓
[4] MERGE FINDINGS — dedupe by (file, line, rule_id)
   ↓
[5] ADOPTION GATE — check trigger conditions
   │ YES → discover → curate → filter
   │ NO  → skip (~90% of audits)
   ↓
[6] VERDICT COMPUTATION
   • severity_score = weighted sum of findings
   • architectural_triggers (god_object, circular_ownership, etc.)
   ↓
[7] SYNTHESIZE REPORT — render via templates/report.md
   ↓
Output: reports/history/YYYY-MM-DD/codebase_analysis.md
```

## Verdict Model (Two-Dimensional)

**Severity score:**
```
score = (critical × 10) + (major × 5) + (minor × 2) + (advisory × 1) + (suggestion × 0.5)
```

**Architectural triggers (any → REDESIGN):**
- `god_object` — single class with excessive responsibilities
- `circular_ownership` — cycles in ownership graph
- `multiple_sources_of_truth` — same state owned by ≥2 components
- `state_duplication` — mirrored state across boundaries
- `hidden_mutation` — non-obvious side effects
- `cyclic_dependency` — module-level cycles

**Verdict mapping:**

| Verdict | Trigger |
|---|---|
| PASS | score=0 AND no triggers |
| PATCH | score≤10 AND no triggers |
| REFACTOR | 10<score≤30 AND no triggers |
| MAJOR | score>30 AND no triggers |
| REDESIGN | any trigger OR score>50 |

## Vendor Neutrality

Boilerplater is vendor-agnostic. The skill is markdown. The rules are pure YAML. No vendor-specific syntax. Usage examples in `README.md` cover Claude Code, Gemini, ChatGPT, Codex, Cursor, Roo, Cline — no first-class vendor.

## Backward Compatibility

- `skills/jreng_compliance/SKILL.md` → symlink alias to canonical orchestrator
- `SKILL.v0-deprecated.md` preserved with deprecation header
- README deprecation notice + vendor-agnostic usage section

## Non-Goals (v1)

- Performance auditor with audio-thread rules (deferred to v1.1)
- Curated Adoption as separate repository (deferred to v2+)
- Real-time/incremental audit
- Non-C++ language support
- Capability implementations (capabilities/ directory exists but not loaded)

## Future Extensions (v1.1+)

- Audio-thread performance rules (`rules/audio-thread/`)
- DSP-specific rules (`rules/dsp/`)
- Capabilities: SARIF export, LSP integration, vector cache
- Adoption repo extraction when bounded context matures

---

*Last updated: 2026-06-28*