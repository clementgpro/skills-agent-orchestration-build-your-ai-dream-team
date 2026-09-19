# Agent team

To build Mona's Project Pulse dashboard, I'm using GitHub Copilot CLI in a Codespace to orchestrate a team of custom agents defined under `.github/agents/`.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases based on file ownership and dependencies, delegates work, verifies the integrated result, and reports progress. Does not implement work itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase, documentation, dependencies, and edge cases to produce an implementation plan (ordered steps, file assignments, dependencies, parallelizable work, validation expectations, open questions). Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and interaction flow. For Project Pulse, produces a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks (`.dashboard`, `.project-card`).
- **Definition:** `.github/agents/designer.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code-oriented tasks (structure, logic, error handling) within the file scope assigned by the Orchestrator. For Project Pulse, also creates supporting run/preview config such as `.vscode/launch.json`.
- **Definition:** `.github/agents/coder.agent.md`

## Orchestration note

All of this work is orchestrated using GitHub Copilot CLI running in a Codespace, which coordinates the Planner, Designer, and Coder agents to deliver Mona's Project Pulse dashboard.
