# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent team defined under `.github/agents/`, orchestrated with GitHub Copilot CLI running in a Codespace.

## Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer agents. Breaks the request into phases with explicit, non-overlapping file scopes, runs independent work in parallel, and reports final outcomes. Does not implement work itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the repository and relevant docs, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder

- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements code within the file scope assigned by the Orchestrator, following existing repository patterns with clear, deterministic, testable behavior. For Project Pulse, also creates supporting files like `.vscode/launch.json` when assigned, configured to launch from the `app` folder and open `index.html`.
- **Definition:** `.github/agents/coder.agent.md`

## Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, information architecture, and visual design within its assigned scope. For Project Pulse, delivers a polished dashboard with project cards, status badges, priority treatment, and deterministic CSS hooks (`.dashboard`, `.project-card`).
- **Definition:** `.github/agents/designer.agent.md`

All four agents are orchestrated using GitHub Copilot CLI in a Codespace, and none of them stage, commit, or push changes — git operations remain under the learner's control.
