# Boilerplater

This directory serves as the master repository for architectural boilerplates, compliance rules, and automated agent skills.

## Core Philosophy: Read-Only Compliance

The tools and skills in this repository operate on a strictly **read-only and suggest** basis. Agents utilizing Boilerplater skills will not destructively alter or refactor your codebase on their own. Instead, they will:
1. Scan the codebase for compliance violations.
2. Generate comprehensive markdown artifacts (e.g., `codebase_analysis.md`).
3. Detail clear, step-by-step suggestions for replacing hacky code with standardized boilerplates.

## Contents

- `skills/`: Contains custom agent skills (e.g., `jreng_compliance_checker`) that enforce our architectural contracts across different projects.

## Usage

Skills in this directory are registered in the global Antigravity configuration via `~/.gemini/config/skills.json`. This ensures that any agent operating in any workspace can load and execute these skills when requested.

## Model Compatibility & Performance

Because Boilerplater and its skills are written in plain markdown, the logical rules are universally understandable by most advanced LLMs. However, for the **best execution performance**—specifically regarding the "Curated Adoption" web-searching capabilities—it is highly recommended to run these skills using **Google models (e.g., Gemini Pro/Flash)** within the Antigravity framework.

- **Native Tool Integration:** The instructions to use the `search_web` tool hook directly into Gemini's native agent capabilities, allowing it to rapidly query the live web and pull down GitHub repos without hallucinating.
- **Context Windows:** Processing the strict compliance documents alongside a large codebase and external GitHub data requires a massive context window. Gemini is uniquely optimized to hold all these strict rules in memory simultaneously while parsing external code.
