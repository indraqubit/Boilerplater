# Boilerplater — Implementation Plan

**Version:** 1.0
**Date:** 2026-06-28
**Source SPEC:** `SPEC.md` (approved 2026-06-28)
**Status:** ⬜ DRAFT → awaiting ARCHITECT approval → ✅ APPROVED

---

## Scope

Implement Boilerplater v1 per approved SPEC. Decomposed into 6 sequential phases. Each phase has a verifiable exit gate. No phase starts until the previous phase's gate passes.

Total estimated deliverable: 24+ rule YAML files, 9 skill markdown files, 4 templates, 1 sample report, contract abstraction layer, capability extension stub, rename + alias migration.

---

## Phase Tracker

| Phase | Status | Description |
|-------|--------|-------------|
| 1 | ⬜ TODO | Foundation structure + contracts/ abstraction + reports/ + capabilities/ stub |
| 2 | ⬜ TODO | Rule Engine: 24+ YAML rule files across 4 domains with metadata |
| 3 | ⬜ TODO | Orchestrator + 4 specialists + adoption/ subagent |
| 4 | ⬜ TODO | Templates (report, patch, refactor, redesign) + sample report |
| 5 | ⬜ TODO | Rename + backward compatibility (alias, README deprecation) |
| 6 | ⬜ TODO | Documentation pass + ARCHITECTURE.md + validation against §8 |

---

## Phase 1 — Foundation Structure

**Objective:** Establish the new directory tree per SPEC §4.1 without committing any rule content yet.

**Tasks:**

1.1. Create `contracts/` directory with three symlinks:
- `contracts/JRENG-CODING-STANDARD.md` → `../../Users/indraqadarsih/Documents/carol-main/JRENG-CODING-STANDARD.md`
- `contracts/MANIFESTO.md` → `../../Users/indraqadarsih/Documents/carol-main/MANIFESTO.md`
- `contracts/NAMES.md` → `../../Users/indraqadarsih/Documents/carol-main/NAMES.md`

1.2. Create `rules/{language,blessed,names,layout}/` directories (empty for now).

1.3. Create `skills/compliance/` and `skills/adoption/` directories (empty for now).

1.4. Create `capabilities/` directory and write `capabilities/README.md` defining the capability registration contract (what a capability file must contain, how it's loaded, that v1 does not load any).

1.5. Create `templates/` directory (empty for now).

1.6. Create `reports/` directory with:
- `reports/.gitignore` — excludes `latest.json` and `history/*` from git
- `reports/history/.gitkeep`
- `reports/latest.json` — placeholder file with `{"baseline": null}` schema

1.7. Create `examples/` directory (empty for now).

**Exit Gate:**
- [ ] `ls contracts/` returns the three symlink names
- [ ] `readlink contracts/JRENG-CODING-STANDARD.md` resolves to a readable file
- [ ] `find rules -type d` returns 4 directories (`language`, `blessed`, `names`, `layout`)
- [ ] `find skills -type d` returns 2 directories (`compliance`, `adoption`)
- [ ] `find capabilities -name '*.md'` returns exactly one file (`README.md`)
- [ ] `cat reports/.gitignore` lists `latest.json` and `history/`
- [ ] `cat reports/latest.json` is valid JSON with `baseline` field
- [ ] No rule YAML files exist yet (deferred to Phase 2)

---

## Phase 2 — Rule Engine

**Objective:** Author 24+ rule YAML files (minimum per domain) using the schema in SPEC §4.3.

**Tasks:**

2.1. Write 7 language rules in `rules/language/`:
- `lang-no-early-return.yaml` (severity: critical)
- `lang-positive-nesting.yaml` (severity: critical)
- `lang-no-anonymous-namespace.yaml` (severity: major)
- `lang-fail-fast.yaml` (severity: critical)
- `lang-use-at-not-subscript.yaml` (severity: major)
- `lang-no-magic-numbers.yaml` (severity: minor)
- `lang-no-raw-delete.yaml` (severity: critical)

2.2. Write 7 BLESSED rules in `rules/blessed/`:
- `blessed-explicit-encapsulation.yaml` (severity: critical)
- `blessed-single-source-of-truth.yaml` (severity: critical)
- `blessed-stateless.yaml` (severity: critical)
- `blessed-deterministic.yaml` (severity: critical)
- `blessed-lean.yaml` (severity: major)
- `blessed-bound.yaml` (severity: critical)
- `blessed-encapsulation.yaml` (severity: critical)

2.3. Write 7 NAMES rules in `rules/names/`:
- `names-verb-noun-functions.yaml` (severity: advisory, default)
- `names-no-type-encoding.yaml` (severity: **major**, override per OQ-2)
- `names-cognitive-load.yaml` (severity: advisory)
- `names-no-getter-prefix.yaml` (severity: advisory)
- `names-no-helper-suffix.yaml` (severity: advisory, default — per OQ-2)
- `names-domain-terms.yaml` (severity: advisory)
- `names-no-comments-needed.yaml` (severity: advisory)

2.4. Write 3 layout rules in `rules/layout/`:
- `layout-includes-order.yaml` (severity: minor)
- `layout-brace-style.yaml` (severity: minor)
- `layout-line-length.yaml` (severity: minor)

2.5. Schema validation pass:
- Every rule has `schema_version: 1`, `rule_version: 1.0`, `status: stable` (or `experimental`)
- Every rule has `id` matching `^{domain}-{kebab-case}$`
- Every rule has `severity_default`, `confidence_floor`, `source_refs`
- Every `source_refs` path begins with `contracts/`
- All rule IDs globally unique across domains
- Every rule has `tags` (array, may be empty) and `depends_on` (array, may be empty)

**Exit Gate:**
- [ ] `find rules -name '*.yaml' | wc -l` returns ≥ 24
- [ ] `find rules -name '*.yaml' | xargs grep -l '^id: ' | wc -l` equals total rule count (every rule has an id)
- [ ] `grep -h '^id: ' rules/**/*.yaml | sort | uniq -d` returns empty (no duplicate IDs)
- [ ] `grep -h '^id: ' rules/**/*.yaml | grep -E '^(lang|blessed|names|layout)-' | wc -l` equals total rule count (all IDs have prefix)
- [ ] `grep -L '^schema_version:' rules/**/*.yaml` returns empty
- [ ] `grep -L '^rule_version:' rules/**/*.yaml` returns empty
- [ ] `grep -L '^status:' rules/**/*.yaml` returns empty
- [ ] `grep '^severity_default:' rules/names/names-no-type-encoding.yaml` shows `major`
- [ ] `grep '^severity_default:' rules/names/names-no-helper-suffix.yaml` shows `advisory`
- [ ] `grep -r 'carol-main' rules/` returns empty (no raw carol-main references)
- [ ] All `source_refs` paths verified to start with `contracts/`

---

## Phase 3 — Skills (Orchestrator + Specialists + Adoption)

**Objective:** Write all skill markdown files. Orchestrator must perform dynamic domain discovery (no hardcoded domain list).

**Tasks:**

3.1. Write `skills/compliance/audit.md` (the `compliance_orchestrator`):
- Defines the audit flow per SPEC §4.2
- Step 1 (DISCOVER DOMAINS): must scan `rules/*` directories at runtime — verified by absence of hardcoded domain list
- Step 2 (LOAD RULES): must validate schema, reject malformed rules
- Step 3 (PARALLEL DISPATCH): must invoke specialists by discovered domain name
- Steps 4–7 per SPEC

3.2. Write `skills/compliance/language.md`:
- Specialist for language rules
- Receives rule list, scans source files, emits findings

3.3. Write `skills/compliance/blessed.md`:
- Specialist for BLESSED principles
- Focus on architectural shape, not syntax

3.4. Write `skills/compliance/names.md`:
- Specialist for naming rules
- Default severity treatment per SPEC §4.5

3.5. Write `skills/compliance/layout.md`:
- Specialist for layout rules
- Lowest priority dispatch

3.6. Write `skills/adoption/discover.md`:
- External search skill (search_web invocation)
- Input: violation context
- Output: candidate boilerplate list

3.7. Write `skills/adoption/curate.md`:
- Rewrites candidates to match project constraints
- Applies JRENG/BLESSED/NAMES filters

3.8. Write `skills/adoption/filter.md`:
- Final pass filter using project rules
- Returns only BLESSED-compliant candidates

**Exit Gate:**
- [ ] `find skills -name '*.md' | wc -l` returns 9 (1 orchestrator + 4 specialists + 3 adoption + 1 deprecated alias)
- [ ] `grep -i 'language.*blessed.*names.*layout' skills/compliance/audit.md` returns empty (no hardcoded domain sequence)
- [ ] `grep -i 'discover.*domain\|scan.*rules' skills/compliance/audit.md` shows dynamic discovery language
- [ ] Each specialist file (`language.md`, `blessed.md`, `names.md`, `layout.md`) exists and is non-empty
- [ ] Each adoption file (`discover.md`, `curate.md`, `filter.md`) exists and is non-empty
- [ ] Adoption trigger guard documented in `audit.md` (verify: grep for "trigger conditions" or "adoption" near "invoke" or "skip")
- [ ] No skill file references `carol-main` directly (must go through `contracts/`)

---

## Phase 4 — Templates & Sample Report

**Objective:** Author output templates and a sample audit report demonstrating the format.

**Tasks:**

4.1. Write `templates/report.md`:
- Skeleton per SPEC §5
- Sections: Summary, Findings by Severity, Findings Detail, Adoption Suggestions (conditional), Baseline Delta, ROI Estimate

4.2. Write `templates/patch.md`:
- Verdict=PATCH specific template
- Smaller, actionable section

4.3. Write `templates/refactor.md`:
- Verdict=REFACTOR/MAJOR template
- Includes architectural breakdown

4.4. Write `templates/redesign.md`:
- Verdict=REDESIGN template
- Emphasizes architectural triggers

4.5. Write `examples/sample-report.md`:
- Synthetic audit output showing all sections filled
- Demonstrates severity scoring, architectural triggers, adoption conditional, baseline delta

**Exit Gate:**
- [ ] `find templates -name '*.md' | wc -l` returns 4
- [ ] `grep -l 'Verdict' templates/*.md` returns 4 files
- [ ] `grep -l 'Summary' templates/*.md` returns ≥ 1 file
- [ ] `examples/sample-report.md` exists and is non-empty
- [ ] Sample report includes severity score calculation example
- [ ] Sample report shows both severity-only and architectural-trigger paths to REDESIGN

---

## Phase 5 — Rename & Backward Compatibility

**Objective:** Migrate from `jreng_compliance_checker` to `compliance_orchestrator` with alias support.

**Tasks:**

5.1. Create `skills/jreng_compliance/SKILL.md` as a symlink:
- `skills/jreng_compliance/SKILL.md` → `../compliance/audit.md`

5.2. Update `README.md`:
- Add deprecation notice section near top
- Point to `skills/compliance/audit.md` as the new canonical path
- Document the v1.x → v2.0 alias window
- Add vendor-agnostic usage examples (Claude Code, Gemini, ChatGPT, Codex — non-exhaustive, non-first-class)

5.3. Verify alias resolves:
- Both `skills/compliance/audit.md` and `skills/jreng_compliance/SKILL.md` point to the same orchestrator

**Exit Gate:**
- [ ] `readlink skills/jreng_compliance/SKILL.md` resolves to a readable file
- [ ] `diff <(readlink skills/jreng_compliance/SKILL.md | xargs cat) skills/compliance/audit.md` returns empty (identical content via alias)
- [ ] `grep -i 'deprecat' README.md` returns ≥ 1 match
- [ ] `grep -i 'compliance_orchestrator\|jreng_compliance_checker' README.md` shows both names mentioned with migration guidance
- [ ] `grep -c 'Claude Code\|Gemini\|ChatGPT\|Codex' README.md` returns ≥ 2 (multi-vendor examples)

---

## Phase 6 — Documentation & Validation

**Objective:** Document the new architecture and validate against SPEC §8 acceptance criteria.

**Tasks:**

6.1. Write `ARCHITECTURE.md`:
- Descriptive map of v1 components per SPEC
- Section: Domain Discovery (orchestrator scans rules/*)
- Section: Rule Engine (24+ rules, metadata schema)
- Section: Specialist Dispatch (parallel by domain)
- Section: Adoption Gate (conditional subagent)
- Section: Verdict Computation (two-dimensional)
- Section: Report Storage (user-managed, latest.json symlink)
- Section: Contract Abstraction (contracts/ indirection)
- Section: Capabilities Extension (capabilities/ reserved)

6.2. Run all SPEC §8 acceptance criteria as verification:
- §8.1 Repository Structure
- §8.2 Rule Engine
- §8.3 Orchestrator Behavior
- §8.4 Backward Compatibility
- §8.5 Vendor-Agnostic
- §8.6 Contract Abstraction
- §8.7 Capabilities Extension
- §8.8 Spec Traceability

6.3. Append to `carol/SPRINT-LOG.md` per CAROL.md protocol:
- Sprint number (next available)
- Agents participated
- Files modified (path:line format)
- Alignment check (BLESSED, NAMES, MANIFESTO)
- Debts paid (none — clean v1)
- Debts deferred (none)

6.4. Hygiene: no debts added since this is a clean refactor; DEBT.md untouched.

**Exit Gate:**
- [ ] `ARCHITECTURE.md` exists at repo root and references every component in SPEC §4
- [ ] All SPEC §8.1–§8.7 checkboxes pass (executed as commands, results logged)
- [ ] `carol/SPRINT-LOG.md` has new entry at top
- [ ] SPRINT-LOG entry references all files created/modified with line numbers
- [ ] No commit performed (ARCHITECT runs git per CAROL.md)

---

## Sequencing Rationale

Phases execute strictly in order because:

1. **Phase 1 (Foundation)** establishes paths that later phases depend on. `contracts/` must exist before rules can reference it. `rules/` must exist before Phase 2 content. `skills/` must exist before Phase 3 content.

2. **Phase 2 (Rules)** must precede Phase 3 (Skills) because specialists need to reference rule schemas and IDs. Without rules, the orchestrator has nothing to dispatch.

3. **Phase 3 (Skills)** must precede Phase 4 (Templates) because templates consume the finding format produced by skills.

4. **Phase 4 (Templates)** must precede Phase 5 (Rename) because the rename affects documentation that references the canonical orchestrator path.

5. **Phase 5 (Rename)** must precede Phase 6 (Validation) because validation checks include the alias resolution.

No parallelization is safe across these phases. Each phase's exit gate proves the prior phase's contracts hold.

---

## Out of Plan

These are explicitly NOT in this plan:
- Rule content changes after v1 (handled via `rule_version` bumps)
- Adding new domains like `dsp/` or `audio-thread/` (handled by extension per G8 — no orchestrator change needed)
- Capabilities implementation (`github-search`, `sarif`, `lsp`, etc. — directory stub only)
- Performance auditor expansion (deferred to v1.1 per OQ-9)
- Adoption repo extraction (deferred to v2+ per ARCHITECT decision)

---

## Approval

**Status:** ⬜ DRAFT → awaiting ARCHITECT approval → ✅ APPROVED

**Approver:** ARCHITECT
**Date:** ____

Once approved, COUNSELOR delegates to @Engineer per phase, @Auditor validates per phase exit gate. PLAN execution proceeds without per-step ARCHITECT round-trip per CAROL.md Step Gate protocol.