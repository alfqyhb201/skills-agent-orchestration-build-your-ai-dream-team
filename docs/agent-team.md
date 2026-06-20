# Agent team

This repository defines a small custom agent team for building Mona's Project Pulse dashboard. Below is each agent, its model, responsibility, and the agent definition path.

- Orchestrator — model: Claude Opus 4.7 (copilot)
  - Responsibility: coordinate Planner, Coder, and Designer; decompose requests into phases, assign explicit file scopes, run tasks in parallel or sequentially as needed, and verify integrated results.
  - Definition: .github/agents/orchestrator.agent.md

- Planner — model: Claude Opus 4.7 (copilot)
  - Responsibility: research the codebase and dependencies, produce an ordered implementation plan with file assignments, dependencies, edge cases, and validation expectations.
  - Definition: .github/agents/planner.agent.md

- Coder — model: GPT-5.5 (copilot)
  - Responsibility: implement code and runnable app support (e.g., create deterministic `.vscode/launch.json` for Project Pulse), validate changes, and keep behavior testable and deterministic.
  - Definition: .github/agents/coder.agent.md

- Designer — model: Gemini 3.1 Pro (copilot)
  - Responsibility: UI/UX, accessibility, information architecture, visual design, and responsive dashboard styling (use deterministic CSS hooks like `.dashboard` and `.project-card`).
  - Definition: .github/agents/designer.agent.md

All work is coordinated via the GitHub Copilot CLI running in this Codespace; the Orchestrator delegates tasks to the specialist agents and reports progress and integration status.