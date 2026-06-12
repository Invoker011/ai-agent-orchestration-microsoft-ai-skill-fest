# Agent team

Below is the custom agent team used to build Mona's Project Pulse dashboard:

- Orchestrator — model: Claude Opus 4.7. Role: coordinates Planner, Coder, and Designer; breaks work into phases and delegates. Definition: .github/agents/orchestrator.agent.md
- Planner — model: Claude Opus 4.7. Role: researches the codebase, identifies edge cases, and produces an implementation plan with file assignments and dependencies. Definition: .github/agents/planner.agent.md
- Coder — model: GPT-5.5. Role: implements code, fixes bugs, and creates runnable app support (e.g., .vscode/launch.json) within the Orchestrator-assigned file scope. Definition: .github/agents/coder.agent.md
- Designer — model: Gemini 3.1 Pro. Role: UI/UX, accessibility, information architecture, and visual design for Project Pulse. Definition: .github/agents/designer.agent.md

This work is orchestrated using the GitHub Copilot CLI running in a Codespace.
