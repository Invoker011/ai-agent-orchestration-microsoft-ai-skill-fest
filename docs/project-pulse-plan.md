# Project Pulse Implementation Plan

Summary
A lightweight, static Project Pulse dashboard that helps contributors quickly see which projects are active, who owns them, status, recent activity, priority/risk and a short summary. Deliverables are a small static app served from app/ with three main files (app/index.html, app/styles.css, app/project-data.json) plus a deterministic VS Code launch configuration (.vscode/launch.json and supporting .vscode/tasks.json) so the dashboard opens directly in the browser from the Codespace/VS Code Run panel. The Orchestrator will coordinate Planner, Designer and Coder work; Designer delivers polished CSS with deterministic hooks (.dashboard, .project-card, etc.); Coder implements HTML, data wiring, and launch tooling; Planner (this document) sequences and validates the work.

## Phase 0 — Preflight (Planner) — (Planner)
Goal: Confirm constraints, file ownership, and initial decisions so phases do not conflict.
Files created/touched by Planner:
- docs/project-pulse-plan.md (this file)

Tasks
- Produce this implementation plan and publish to docs/project-pulse-plan.md.
- Confirm minimal runtime: use a simple HTTP server (python -m http.server) to serve app/ to avoid file:// fetch restrictions in modern browsers.
- Confirm deterministic port: 8000 (configurable via Open Questions below).

Validation
- Plan reviewed and approved by Orchestrator before work begins.

Dependencies
- None. This is the plan source of truth.

---

## Phase 1 — Design & CSS Hooks (Designer) — (Designer)
Goal: Produce a polished CSS file and explicit design contract (class/ID hooks) the Coder will implement against.

Files Designer will create or modify:
- app/styles.css
- docs/project-pulse-design.md (short design notes — optional but recommended)

Designer deliverables and expectations (exact, deterministic):
- Provide a ready-to-use CSS file `app/styles.css` implementing the Project Pulse visual design, meeting the UX expectations listed below.
- Provide a short design notes doc listing all CSS hooks and responsive breakpoints.

Deterministic CSS hooks (required)
- .dashboard — the top-level container for the dashboard
- .dashboard__header — header area (title, summary)
- .dashboard__grid — card grid wrapper (responsive layout)
- .project-card — single project card
- .project-card__title — project title
- .project-card__meta — owner/status/priority line
- .project-card__status — status badge (use data-status or class modifiers)
- .project-card__priority — priority badge (use data-priority or class modifiers)
- .project-card__activity — recent activity / summary text
- .badge--status-{status} — status variants, e.g., badge--status-active, badge--status-paused, badge--status-complete
- .badge--priority-{level} — priority variants, e.g., badge--priority-high, badge--priority-medium, badge--priority-low
- .sr-only — accessibility utility for screen readers

CSS / UX expectations (explicit)
- Visual affordances: rounded corners, subtle drop shadow, readable type scale, padding and consistent spacing.
- Status badges should be visually distinct (color and accessible contrast) and use the deterministic hook `.project-card__status`.
- Priority should be emphasized (color or icon) and use `.project-card__priority`.
- Responsive grid:
  - Mobile width (<600px): single column
  - Tablet (600–900px): two columns
  - Desktop (>=900px): three columns
- Accessible font sizes, minimum 16px body, 1.4 line-height for summaries.
- Truncation rules for long text: title max 2 lines with ellipsis; summary max 3 lines; ensure keyboard focus styles for cards.
- Ensure sufficient color contrast for badges and text per WCAG AA where feasible.
- Provide light-weight utility variables at top for colors and spacing to keep the file readable.

Designer notes to include:
- Provide class names exactly as above and examples of class modifiers (badges).
- Provide minimal CSS reset or rely on simple base rules in styles.css.

Files NOT to touch:
- app/index.html (unless Orchestrator explicitly assigns the Designer to modify markup — keep markup changes to Coder).
- app/project-data.json

Validation
- Verify app/styles.css exists and contains the hooks above (search for the classes).
- Quick manual check: styles define grid responsive breakpoints and badge variants.
- Accessibility spot checks: color contrast and keyboard focus styles.

Dependencies
- Designer output must be available before Coder finalizes markup or styling expectations. (See sequential constraint.)

Why sequential here:
- Coder needs the exact class hooks (names) to mark up HTML; to avoid rework, Designer should publish hooks and a short sample markup snippet before Coder begins implementing index.html.

Estimated effort
- Small (1–2 hours).

---

## Phase 2 — Data & HTML Skeleton (Coder) — (Coder)
Goal: Implement the interactive, data-driven HTML page that consumes app/project-data.json and styles it using app/styles.css. Also add deterministic launch configuration so the learner can run the dashboard in Codespace.

Files Coder will create or modify:
- app/index.html (create/implement)
- app/project-data.json (create sample dataset)
- .vscode/launch.json (create)
- .vscode/tasks.json (create) — to run the local HTTP server prior to opening the browser (preLaunch)
- optionally, docs/project-pulse-usage.md (short run instructions — optional)

Coder responsibilities (explicit)
- Implement `app/index.html`:
  - Load `app/styles.css` via a relative <link>.
  - Use the deterministic CSS hooks produced by Designer (Planner ensures Designer provides them).
  - Provide a minimal, semantic HTML structure:
    - header with .dashboard__header
    - main with .dashboard and .dashboard__grid
  - Implement an accessible project-card template within index.html, and wire a small client-side script (vanilla JS) that:
    - Fetches `/project-data.json` (relative path) using fetch().
    - Handles fetch errors with a clear UI message and console logs.
    - Renders cards dynamically into `.dashboard__grid`.
    - Adds accessible attributes (role="list", role="listitem" or article tags) and keyboard focusability for cards.
    - Provides simple client-side filtering or no JS option fallback (progressive enhancement).
- Implement `app/project-data.json`:
  - Provide a top-level `projects` array with at least 4 sample projects.
  - Each project object must include: name, owner, status, recentActivity, priority.
  - Provide variations for status (e.g., "active", "paused", "complete") and priority ("high","medium","low").
- Implement `.vscode/tasks.json`:
  - Define a task named "Run Project Pulse HTTP Server" that runs:
    - "python -m http.server 8000" with the working directory set to `${workspaceFolder}/app` (strict JSON).
  - Use `isBackground: true` if needed and make sure it is compatible with Codespaces.
- Implement `.vscode/launch.json`:
  - Strict JSON (no comments).
  - Create a deterministic configuration named "Run Project Pulse Dashboard".
  - Use a browser debugger launch (e.g., "type": "pwa-chrome" or "pwa-msedge") with:
    - "request": "launch"
    - "name": "Run Project Pulse Dashboard"
    - "url": "http://localhost:8000/index.html"
    - "cwd": "${workspaceFolder}/app"
    - "preLaunchTask": "Run Project Pulse HTTP Server"
  - This ensures the app is served from app/ and the browser opens `index.html`.
- Validation steps (below).

Why serve with HTTP:
- Fetching local JSON via fetch() is blocked in many browsers when loading index.html with file://. Serving over HTTP (python -m http.server) is reliable and simple across Codespaces.

Files NOT to touch:
- app/styles.css (Designer owns styling)
- docs/project-pulse-plan.md (Planner owns plan)

Validation (detailed)
- After implementing files, run the VS Code Run panel:
  - Select "Run Project Pulse Dashboard" and Start.
  - Expectation: preLaunchTask starts Python server in `${workspaceFolder}/app`, listening on port 8000.
  - The browser debug session opens http://localhost:8000/index.html and shows the dashboard UI.
- Manual smoke tests:
  - Confirm at least 4 project cards render.
  - Confirm each card shows name, owner, status, priority, and recentActivity.
  - Confirm status and priority badges have appropriate CSS classes (badge--status-* and badge--priority-*).
  - Turn off the server (stop task) and verify fetch error is handled (UI shows message like "Could not load project data").
- File content checks:
  - app/project-data.json contains top-level `projects` array with objects formatted as:
    {
      "projects": [
        { "name": "...", "owner": "...", "status": "...", "recentActivity": "...", "priority": "..." },
        ...
      ]
    }
- Test responsive layout by resizing the browser:
  - Mobile single column, tablet two, desktop three.
- Accessibility checks:
  - Tab through cards; each card is focusable and focus outline is visible.
  - Badge text is readable and contrasts with background.

Edge-case testing for Coder
- Fetch returns 404 — show friendly error.
- Malformed JSON — catch JSON.parse error and show friendly error.
- Missing fields (owner/status/priority) — render "Unknown" or fallback text.
- Long text wrapping/truncation — confirm ellipsis rules and accessible tooltip on hover for truncated text (optional).

Dependencies
- Must wait for Designer Phase to publish the final class hook names (or a minimal agreed list) before finalizing index.html markup. However, Coder may start creating project-data.json in parallel.

Estimated effort
- Medium (2–4 hours).

---

## Phase 3 — Integration, QA, and Polishing (Orchestrator + Coder + Designer)
Goal: Validate integrated app end-to-end, fix small layout issues, tune CSS, and produce final documentation for the learner.

Files touched:
- app/index.html (Coder: tweaks)
- app/styles.css (Designer: tweaks)
- .vscode/launch.json (Coder: final check)
- app/project-data.json (Coder: adjust sample data)
- docs/project-pulse-usage.md (optional: Steps to run & test)

Tasks and responsibilities
- Orchestrator: coordinate integration, run the Run config, assign fixes.
- Coder: fix JS edge cases, error handling, and ensure launch config works in Codespace.
- Designer: adjust spacing, color contrast, mobile layout refinements.
- Optional: add a lightweight smoke-test script or checklist in docs.

Validation
- Full run in Codespace:
  - Start "Run Project Pulse Dashboard".
  - Confirm the debug browser opens and shows dashboard with data.
- Cross-check CSS rule usage — ensure no orphan hooks.
- Accessibility spot-checks and color contrast adjustments.

Parallelism
- Small polishing tasks that do not overlap file ownership can run in parallel:
  - Coder adjusts JS error handling while Designer tweaks CSS colors.
  - Orchestrator runs tests and collates issues.

Dependencies
- Final integration requires both HTML and CSS to be in place.

Estimated effort
- Small (1–2 hours).

---

## Files (explicit list) and owner assignments

Phase 0 (Planner)
- docs/project-pulse-plan.md — Planner (this document)

Phase 1 (Designer)
- app/styles.css — Designer (create + deliver)
- docs/project-pulse-design.md — Designer (optional notes & hooks list)

Phase 2 (Coder)
- app/index.html — Coder (create and implement markup + client JS)
- app/project-data.json — Coder (create sample data)
- .vscode/launch.json — Coder (create deterministic launch config)
- .vscode/tasks.json — Coder (create task that runs python server)
- docs/project-pulse-usage.md — Coder (optional run instructions)

Phase 3 (Integration)
- All of the above as needed for fixes:
  - app/index.html — Coder (tweak)
  - app/styles.css — Designer (tweak)
  - .vscode/launch.json — Coder (validate)

Notes on file ownership
- Keep single-owner per file where possible to avoid merge conflicts: Designer = styles.css, Coder = index.html & project-data.json & .vscode/*, Planner = docs/*.

---

## Dependencies between phases and files

- Phase 1 (Design) -> Phase 2 (Coder): Designer must publish class hook names and a working styles.css before final index.html markup is frozen. This avoids rework.
- Phase 2 (Data & HTML) can start on some items (project-data.json) while Designer finishes styles.css.
- .vscode/launch.json and .vscode/tasks.json depend on Coder decisions but do not depend on Designer; can be created in parallel with Designer work after hooks are agreed.
- Phase 3 (Integration) depends on both 1 and 2 completing.

Which tasks must be sequential:
- Designer publishing CSS hooks must precede final markup decisions (sequential).
- The final integration QA must wait for both CSS and HTML/JS to be in place (sequential).

Which tasks can run in parallel:
- Coder can author `app/project-data.json` in parallel with Designer creating `app/styles.css` (non-overlapping files).
- Coder can draft `.vscode/launch.json` and `.vscode/tasks.json` in parallel once the decision to use HTTP server (port 8000) is agreed.
- Minor style tweaks and small JS improvements during Phase 3 can run in parallel as long as they touch their respective files.

Why these choices
- Non-overlapping file scopes reduce merge conflicts and allow parallel productivity.
- The markup must reflect CSS hooks to prevent mismatches, so design→markup is a natural dependency.

---

## Parallel work decisions — explicit

Allowed parallel tasks:
- Designer: Create app/styles.css
- Coder: Create app/project-data.json and draft .vscode/launch.json & tasks.json
These are safe because the files are disjoint and do not need final runtime to be present.

Tasks that must not run in parallel:
- Designer changing classes while Coder is concurrently committing index.html with the old class names — avoid by having Designer publish a short "hooks list" immediately; Coder waits for that list to finalize markup.

---

## Validation expectations — step-by-step

Phase 1 validation (Designer)
- File presence:
  - app/styles.css exists.
- Content validation:
  - File contains the class names listed in "Deterministic CSS hooks".
  - CSS contains responsive breakpoints for the grid.
  - CSS includes badge variants `.badge--status-*` and `.badge--priority-*`.
- Quick visual check: use browser devtools to confirm expected rules are applied (after Phase 2).

Phase 2 validation (Coder)
- File presence:
  - app/index.html, app/project-data.json, .vscode/launch.json, .vscode/tasks.json exist.
- Launch validation:
  - In VS Code, open Run panel, select "Run Project Pulse Dashboard", click Start.
  - Expected behavior:
    - Task "Run Project Pulse HTTP Server" starts python server in `${workspaceFolder}/app` on port 8000.
    - Browser opens http://localhost:8000/index.html and displays the dashboard (not directory listing).
- Data and UI validation:
  - At least 4 cards render with the expected fields.
  - Error handling: stop the server, reload page, and confirm a friendly error message displays.
- Responsive validation:
  - Resize browser to confirm 1/2/3 column breakpoints.

Phase 3 validation (Integration)
- Cross-browser check in Codespace (Chrome/Edge debug):
  - Cards render and badges have proper colors.
- Accessibility spot checks:
  - Tab to each card and ensure visible focus
  - Confirm semantic HTML tags or appropriate ARIA roles

Smoke test URLs and commands
- Launch configuration will open:
  - http://localhost:8000/index.html
- To manually run server (if needed):
  - cd app
  - python -m http.server 8000
  - Then open http://localhost:8000/index.html in a browser.

---

## Edge cases, risks and mitigations

1. Fetching project-data.json fails in file:// environment
   - Risk: Browsers block fetch() when page is opened via file://
   - Mitigation: Use `.vscode/tasks.json` to run a simple HTTP server (python -m http.server 8000) and `.vscode/launch.json` to open http://localhost:8000/index.html. Verify python is available in Codespace environment.

2. Port 8000 already in use
   - Risk: preLaunchTask fails to start server
   - Mitigation: Provide a clear error message in preLaunchTask logs and a configurable variable at top of `.vscode/tasks.json`. Open Question: preferred fallback port?

3. Data shape mismatches (missing fields)
   - Risk: app runtime errors or missing UI fields
   - Mitigation: Coder must defensively render default text such as "Unknown" or "—" for missing fields; handle JSON parse errors gracefully.

4. Long text overflow and layout breakage
   - Risk: long names or summaries break card layout
   - Mitigation: Designer must provide truncation rules and CSS ellipsis; Coder should add `title` attributes on truncated elements so full content is readable on hover.

5. Accessibility failures (contrast, keyboard)
   - Risk: fails WCAG
   - Mitigation: Designer to target AA contrast; Coder to add focus styles and semantic markup.

6. Codespace environment lacking Python
   - Risk: `python -m http.server` might not be present
   - Mitigation: Fallback option in docs: use `npx http-server` or launch a "file" protocol fallback (but file fetch issues remain). Open Question: preference for python vs node http-server? Confirm available runtime.

7. Naming mismatch between Designer hooks and Coder markup
   - Risk: CSS not applied
   - Mitigation: Designer to publish a hooks list; Coder to reference hooks exactly; Orchestrator to enforce final check in Phase 3.

8. Browser security policies blocking local debug
   - Risk: Browser blocks fetch on localhost due to mixed content or extension settings
   - Mitigation: Use http://localhost (not https); ensure no CSP interfering.

---

## Tests and validation artifacts to include
- A short manual checklist in docs/project-pulse-usage.md:
  - Steps to run the launch config
  - Expected port and URL
  - Smoke test steps
- Minimal automated check (optional):
  - A tiny Node or shell script to curl http://localhost:8000/project-data.json and confirm a 200 JSON with top-level `projects` array (optional; not required).

---

## Open questions for the Orchestrator / user
1. Preferred server runtime: Should the launch task use Python (`python -m http.server`) or Node (`npx http-server`)? (Python is typically present; Node might be preferred if users want a single toolchain.)
2. Preferred default port if 8000 conflicts?
3. Any brand color / font constraints we should apply to the visual design?
4. Do you want any client-side interactivity beyond rendering (e.g., sort, filter, click to open GitHub issues) in scope now?
5. Is it acceptable to create `.vscode/tasks.json` in addition to `.vscode/launch.json`? (Recommended for preLaunch server start.)
6. Should we include a minimal automated smoke test script in repo (optional)?

---

## Quick implementation checklist (to save in docs/project-pulse-plan.md)
- [ ] Planner: publish docs/project-pulse-plan.md (this file)
- [ ] Designer: create app/styles.css containing:
  - .dashboard and children hooks
  - .project-card and badge variants
  - responsive grid breakpoints
  - accessibility styles (focus, contrast)
- [ ] Designer: publish hooks list (docs/project-pulse-design.md)
- [ ] Coder: create app/project-data.json with top-level `projects` array (>=4 entries)
- [ ] Coder: create app/index.html that:
  - Loads app/styles.css
  - Fetches /project-data.json and renders cards with hooks
  - Provides graceful error handling
- [ ] Coder: create .vscode/tasks.json to run `python -m http.server 8000` (cwd = ${workspaceFolder}/app)
- [ ] Coder: create .vscode/launch.json named "Run Project Pulse Dashboard" (cwd = ${workspaceFolder}/app) that opens `http://localhost:8000/index.html` with preLaunchTask bound
- [ ] Orchestrator: run integration, validate, and assign tweaks
- [ ] Coder + Designer: address integration issues and finalize

If you want, I can now produce the minimal "hooks list" and a sample project-data.json example to hand to the Designer and Coder, or split this plan into Copilot CLI tasks for the Orchestrator to distribute. Which would you like next?
