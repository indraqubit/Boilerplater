# Capabilities — Extension Point

**Status:** Stub. No v1 capability is loaded into the audit pipeline.

## Purpose

`capabilities/` is the architectural extension surface for optional features that augment Boilerplater's audit output. v1 defines the directory and the capability contract; v1 does not implement any capability.

## Capability Contract (v1)

A capability is a single `.md` file in this directory. To register a capability:

1. File name: `<capability-name>.md`
2. File frontmatter (YAML):
   ```yaml
   ---
   name: <capability-name>
   version: 1
   status: stub | experimental | stable
   loaded: false
   ---
   ```
3. File body: declares input contract, output contract, dependencies, and v1 non-implementation note.

## v1 Inventory

| Capability | Status | Loaded |
|---|---|---|
| `github-search` | stub | false |
| `sarif` | stub | false |
| `lsp` | stub | false |
| `semantic-index` | stub | false |
| `vector-cache` | stub | false |

These are planned for v1.1+. Their existence in this directory does NOT affect v1 audit flow.

## Loading Semantics (future)

When `loaded: true` is set, the orchestrator MUST NOT modify its core flow. Capabilities are observational/transformational side-cars that consume findings, not ones that gate dispatch.