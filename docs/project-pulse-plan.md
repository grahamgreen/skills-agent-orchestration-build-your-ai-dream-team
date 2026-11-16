# Project Pulse implementation plan

Prepared by the **Planner** for the **Orchestrator**, based on `.github/project-pulse-brief.md` and the agent definitions in `.github/agents/`.

## Summary

Project Pulse is a small static dashboard: one HTML page that fetches a JSON file and renders a card for each project, styled with a single stylesheet. It is served locally with `python3 -m http.server` from the `app/` directory via a VS Code launch configuration. There is no build step, framework, or backend.

## File assignments

| File | Owner | Purpose |
| --- | --- | --- |
| `app/project-data.json` | Coder | Top-level `projects` array; each project has `name`, `owner`, `status`, `recentActivity`, `priority` (plus a short `summary`). |
| `app/index.html` | Coder (structure guided by Designer) | Page titled "Project Pulse"; links `styles.css`; fetches `project-data.json`; renders one `.project-card` per project inside `.dashboard`; shows status, recentActivity, and priority. |
| `app/styles.css` | Designer | Polished, responsive dashboard styling: `.dashboard` grid, `.project-card` with `border-radius` and `box-shadow`, status and priority badges, accessible contrast and focus styles. |
| `.vscode/launch.json` | Coder | Strict JSON (no comments). Configuration **Run Project Pulse Dashboard** runs `python3 -m http.server 5500` with `cwd` `${workspaceFolder}/app` and a `serverReadyAction` that opens `http://localhost:%s/index.html`. |

## Responsibilities

- **Designer:** decides layout, information hierarchy (name → status → priority → owner → recent activity), badge colors for each status and priority value, responsive breakpoints, and accessibility (semantic landmarks, headings, color is never the only signal, visible focus). Owns `app/styles.css` and specifies the class names that `app/index.html` must use.
- **Coder:** implements the data file, the HTML and rendering script, and the launch configuration using the Designer's class names. Handles loading and error states and validates the result.
- **Orchestrator:** sequences the phases below, passes the Designer's class contract to the Coder, and checks that the integrated result works.

## Ordered steps and dependencies

1. **Design contract (Designer).** Define the class names (`.dashboard`, `.project-card`, `.badge`, `.status-*`, `.priority-*`) and the card layout. *Dependencies:* none.
2. **Data (Coder): `app/project-data.json`.** *Dependencies:* none. The field names are fixed by the brief.
3. **Styles (Designer): `app/styles.css`.** *Dependencies:* step 1.
4. **Markup and rendering (Coder): `app/index.html`.** *Dependencies:* step 1 (class names) and step 2 (data shape).
5. **Launch configuration (Coder): `.vscode/launch.json`.** *Dependencies:* none beyond knowing the files live in `app/`.
6. **Integration check (Orchestrator).** *Dependencies:* steps 2–5.

## Parallel work decisions

- **Can run in parallel:** step 2 (`app/project-data.json`) and step 5 (`.vscode/launch.json`) alongside the Designer's steps 1 and 3. None of these share files.
- **Must run sequentially:** step 4 (`app/index.html`) waits for the Designer's class contract and the data shape. Step 6 runs last.
- **No overlapping scopes:** the Designer never edits `app/index.html` directly, and the Coder never edits `app/styles.css`. This prevents conflicts.

## Edge cases

- `project-data.json` fails to load (e.g. the page is opened via `file://`). Show a readable error message instead of an empty page.
- An empty `projects` array shows an empty-state message.
- A project is missing a field or has an unknown status or priority. Render a fallback ("Unknown") with neutral styling instead of throwing.
- Long names or activity text wrap without breaking the card layout.
- Insert data with `textContent`, never `innerHTML`, to avoid injection from data.
- Port 5500 is already in use: surface the http.server error in the debug console.

## Validation expectations

- `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` both succeed.
- `app/index.html` contains the title "Project Pulse" and references `styles.css` and `project-data.json`.
- With `python3 -m http.server 5500` running in `app/`, `http://localhost:5500/index.html` renders one `.project-card` per project showing status, recentActivity, and priority. There is no directory listing.
- `app/styles.css` contains `.dashboard` and `.project-card` with `border-radius` and `box-shadow`, and the layout reflows to one column on narrow screens.
- **Run Project Pulse Dashboard** appears in VS Code's Run and Debug view and opens the dashboard.

## Open questions

- Should the cards be sortable or filterable by priority? Out of scope for v1 unless requested.
- Is the sample data placeholder content, or should it reflect real team projects?
