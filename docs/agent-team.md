# Project Pulse agent team

Mona's Project Pulse dashboard will be built by a team of four custom agents defined in `.github/agents/`. The learner drives the team from GitHub Copilot CLI and controls all git operations; none of the agents stage, commit, or push.

| Agent | Model | Responsibility | Definition file |
| --- | --- | --- | --- |
| **Orchestrator** | Claude Opus 5.5 | Coordinates the request: gets a plan from the Planner, splits it into phases, delegates with explicit file scopes, and verifies the integrated result. Does not implement work itself. | `.github/agents/orchestrator.agent.md` |
| **Planner** | Claude Opus 5.5 | Researches the repository and produces the implementation plan: ordered steps, file assignments, dependencies, parallel vs. sequential work, edge cases, and validation expectations. Does not write code. | `.github/agents/planner.agent.md` |
| **Designer** | Gemini 3.7 Flash | Owns UI/UX: layout, information hierarchy, accessibility, responsive behavior, and visual polish (project cards, status badges, `.dashboard` / `.project-card` hooks). | `.github/agents/designer.agent.md` |
| **Coder** | GPT-5.5 | Implements the static app files within its assigned scope, creates `.vscode/launch.json` as strict JSON when assigned, and validates the change before reporting. | `.github/agents/coder.agent.md` |

## How the team will build Project Pulse

1. **Orchestrate.** In Copilot CLI, select the **Orchestrator** with `/agent` and give it the Project Pulse brief (`.github/project-pulse-brief.md`).
2. **Plan.** The Orchestrator asks the **Planner** for a plan, saved to `docs/project-pulse-plan.md`, that assigns ownership of `app/index.html`, `app/styles.css`, `app/project-data.json`, and `.vscode/launch.json`.
3. **Design and build.** The Orchestrator delegates visual and accessibility decisions to the **Designer** (`app/styles.css` and the markup structure) and implementation to the **Coder** (`app/index.html`, `app/project-data.json`, `.vscode/launch.json`). Work runs in parallel only where file scopes don't overlap; shared files are handled sequentially.
4. **Validate and hand off.** The Orchestrator checks that the pieces fit together (the page loads the data, renders project cards, and the **Run Project Pulse Dashboard** launch configuration opens `index.html`), then summarizes the result in `docs/final-handoff.md`.
5. **Ship.** The learner reviews each phase and asks Copilot CLI to stage, commit, and push.
