# Boilerplater 🛡️

## ⚠️ Deprecation Notice (v1.0)

The audit skill was renamed in v1.0:
- **Old name:** `jreng_compliance_checker` (deprecated, kept as alias)
- **New name:** `compliance_orchestrator` (canonical)
- **Canonical path:** `skills/compliance/audit.md`
- **Alias path:** `skills/jreng_compliance/SKILL.md` (symlink to canonical)

The alias will be removed in v2.0. Migrate to the new name at your earliest convenience.

---

**Boilerplater** is an automated, agent-driven repository designed to enforce strict architectural standards, manage compliance rules, and systematically reduce technical debt across all your C++/JUCE projects.

Boilerplater acts as a **Read-Only Compliance Auditor**. It scans your codebase, finds architectural violations, and generates comprehensive markdown reports suggesting exactly how to replace "hacky" code with approved boilerplate patterns.

---

## What's New in v1.0

v1.0 transforms the v0 single-skill audit into a **modular multi-agent governance layer**:

- **24 rules** across 4 domains (`language`, `blessed`, `names`, `layout`), each with explicit IDs (`lang-`, `blessed-`, `names-`, `layout-` prefixes)
- **Two-dimensional verdict:** 5-tier scale (`PASS` / `PATCH` / `REFACTOR` / `MAJOR` / `REDESIGN`) computed from severity score + architectural triggers (god_object, circular_ownership, multiple_sources_of_truth, state_duplication, hidden_mutation, cyclic_dependency)
- **Severity tiers:** critical / major / minor / advisory / suggestion with per-rule overrides
- **Confidence scoring:** high / medium / low per finding
- **Dynamic domain discovery:** orchestrator scans `rules/*` at runtime — adding `rules/dsp/` or `rules/audio-thread/` requires no orchestrator changes
- **Conditional adoption subagent:** invoked only on critical/major findings (~10% of audits) — saves context window
- **Contract abstraction:** `contracts/` indirection layer hides CAROL symlink details from rule logic
- **Vendor-agnostic core:** markdown + pure YAML, no proprietary syntax
- **Capabilities extension point:** `capabilities/` reserved for SARIF, LSP, semantic-index, vector-cache

See [SPEC.md](./SPEC.md) for full specification, [ARCHITECTURE.md](./ARCHITECTURE.md) for system map.

---

## How to Use

Boilerplater's audit skill is a markdown file (`skills/compliance/audit.md`). Load it into any LLM tool that can read markdown, point it at your codebase, and the orchestrator takes over.

### Claude Code
```
@skills/compliance/audit.md
```

### Gemini / Antigravity
Load `skills/compliance/audit.md` as context, then run.

### ChatGPT / Codex
Upload or paste `skills/compliance/audit.md`, then run.

### Cursor / Roo / Cline
Reference `skills/compliance/audit.md` from your project's `.cursorrules` or equivalent.

No vendor is "first-class" — all are equally supported. (Legacy note: v0 used Antigravity's `~/.gemini/config/skills.json` registration; this is no longer required.)

### Invocation Example
> *"Run Boilerplater compliance audit on this codebase."*
>
> or
>
> *"Audit this project using `compliance_orchestrator`."*

The agent will discover `skills/compliance/audit.md`, scan your active codebase in strict read-only mode, and write the report to `reports/history/YYYY-MM-DD/codebase_analysis.md`.

---

## The Output Report

The audit report (`reports/history/YYYY-MM-DD/codebase_analysis.md`) contains:

1. **Summary table:** Files scanned, total findings, severity score, architectural triggers fired, verdict, confidence coverage
2. **Findings by severity:** Histogram across critical / major / minor / advisory / suggestion
3. **Findings detail:** For each finding — rule ID, severity, confidence, location, evidence snippet, reason, expected pattern, suggested fix, source refs
4. **Architectural triggers:** god_object, circular_ownership, multiple_sources_of_truth, state_duplication, hidden_mutation, cyclic_dependency (any fires → REDESIGN)
5. **Adoption suggestions:** Only when triggered by critical/major findings (~10% of audits)
6. **Baseline delta:** Comparison against `reports/latest.json` (user-managed baseline pointer)
7. **ROI estimate:** Per-verdict remediation effort estimate

Verdict scale:

| Verdict | Trigger |
|---|---|
| PASS | score=0 AND no triggers |
| PATCH | score≤10 AND no triggers |
| REFACTOR | 10<score≤30 AND no triggers |
| MAJOR | score>30 AND no triggers |
| REDESIGN | any trigger OR score>50 |

---

## Directory Structure

```
Boilerplater/
├── SPEC.md                          # Full specification (v1)
├── PLAN.md                          # Implementation plan (6 phases)
├── ARCHITECTURE.md                  # System map (descriptive)
├── README.md                        # This file
├── Boilerplater.md                  # Master philosophy
├── contracts/                       # Contract abstraction (symlinks to CAROL docs)
│   ├── JRENG-CODING-STANDARD.md
│   ├── MANIFESTO.md
│   └── NAMES.md
├── rules/                           # Rule Engine (24 rules, 4 domains)
│   ├── language/  (7 rules: lang-*)
│   ├── blessed/   (7 rules: blessed-*)
│   ├── names/     (7 rules: names-*)
│   └── layout/    (3 rules: layout-*)
├── skills/
│   ├── compliance/                  # Orchestrator + 4 specialists
│   │   ├── audit.md                 # compliance_orchestrator
│   │   ├── language.md
│   │   ├── blessed.md
│   │   ├── names.md
│   │   └── layout.md
│   ├── adoption/                    # Conditional subagent
│   │   ├── discover.md
│   │   ├── curate.md
│   │   └── filter.md
│   └── jreng_compliance/            # v0 alias (deprecated, removed in v2.0)
│       ├── SKILL.md -> ../compliance/audit.md
│       └── SKILL.v0-deprecated.md
├── capabilities/                    # Extension point (stub)
│   └── README.md
├── templates/                       # Output templates
│   ├── report.md
│   ├── patch.md
│   ├── refactor.md
│   └── redesign.md
├── reports/                         # Audit output (gitignored)
│   ├── latest.json
│   └── history/
└── examples/
    └── sample-report.md
```

---

## How it Works Under the Hood

See [Boilerplater.md](./Boilerplater.md) for the repository's core read-only philosophy, and [ARCHITECTURE.md](./ARCHITECTURE.md) for the system map.
