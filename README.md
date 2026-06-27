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

By leveraging Antigravity Agent Skills, Boilerplater acts as a **Read-Only Compliance Auditor**. It scans your codebase, finds architectural violations, and generates comprehensive markdown reports suggesting exactly how to replace "hacky" code with approved boilerplate patterns.

---

## 🚀 How to Install / Register

To make the Boilerplater skills available to your agent across all workspaces, you need to register this directory in your global Antigravity configuration.

1. Ensure the global config directory exists:
   `~/.gemini/config/`
2. Create or edit `~/.gemini/config/skills.json` to include the absolute path to the `skills/` folder in this repository:
   ```json
   {
     "entries": [
       { "path": "/Users/indraqadarsih/Downloads/TERRACE/Boilerplater/skills" }
     ]
   }
   ```
*(Note: If you move this repository, you must update the path in `skills.json`)*.

---

## 🛠️ How to Run

Because Boilerplater is built on the Antigravity Skills framework, running it is as simple as talking to your AI agent in any of your project workspaces.

**1. Navigate to your project**
Open your target codebase (e.g., `ADDA-M`) in your IDE and launch the Antigravity chat.

**2. Invoke the Agent**
Send a prompt asking the agent to act as the compliance checker. For example:
> *"Run the `jreng_compliance_checker` skill on this project."*
>
> or
>
> *"Please audit my codebase using the Boilerplater compliance agent."*

**3. Review the Output**
The agent will automatically discover the skill instructions from this repository, scan your active codebase (in strict read-only mode), and generate a `codebase_analysis.md` artifact.

---

## Vendor-Agnostic Usage

Boilerplater's core is **vendor-agnostic**. The audit skill is markdown, portable across any LLM tool that reads markdown:

### Claude Code
```
@skills/compliance/audit.md
```

### Gemini
```
Load skills/compliance/audit.md as context, then run.
```

### ChatGPT / Codex
```
Upload or paste skills/compliance/audit.md, then run.
```

### Cursor / Roo / Cline
```
Reference skills/compliance/audit.md from your project's `.cursorrules` or equivalent.
```

No vendor is "first-class" — all are equally supported.

---

## 📄 The Output Report

The generated `codebase_analysis.md` will contain:
1. **Categorized Violations:** Broken down into `JRENG`, `BLESSED`, and `NAMES` rules.
2. **Code Snippets:** A side-by-side comparison of the existing "hacky" code vs. the required "compliant boilerplate."
3. **Curated Adoptions:** If the agent found a novel problem, it will search the web/GitHub, adapt the findings to pass your internal rules, and present them here.
4. **ROI (Return on Investment):** A breakdown of the cognitive load reduced and bugs prevented by the refactoring.
5. **Action Statement:** A final verdict classifying the work as either a **"REDESIGN"** (deep architectural flaws) or a **"5 MINS CHECKLIST DIFF"** (superficial cleanups).

---

## ⚙️ How it Works Under the Hood

See [Boilerplater.md](./Boilerplater.md) for detailed information on the repository's core read-only philosophy, and why executing these tasks on Google's native agentic models provides the highest accuracy and safety.
