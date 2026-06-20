# Mona's Project Pulse — Implementation Plan
Date: 2026-06-20T20:48:59Z

Summary
- Goal: Deliver a lightweight, deterministic, runnable preview of "Mona's Project Pulse" under app/ so a reviewer in the Codespace can open a single URL and see a styled dashboard built from a static JSON file. The preview must be simple, accessible, responsive, and launchable from VS Code using a deterministic .vscode/launch.json.
- Minimal surface: app/index.html, app/styles.css, app/project-data.json, .vscode/launch.json, and .vscode/tasks.json (the latter automates serving the app on port 5000).
- Design constraints: Designer will provide CSS using deterministic hooks such as .dashboard and .project-card. Coder will implement the HTML, data file, minimal JS to render the dashboard, and the deterministic launch & task configurations.

Ordered implementation steps (actionable, numbered)
1. Preflight (Orchestrator)
   - Confirm Codespace environment has python3 available (python3 --version).
   - Confirm port 5000 is allowed/open in Codespace (default Codespace ports are fine).

2. Create project data (Coder) — app/project-data.json
   - Create a deterministic static JSON file at app/project-data.json with a small array (3–6) of projects.
   - Each project object must include these keys: id (string), name (string), owner (string), status (one of "on-track", "at-risk", "blocked"), progress (integer 0–100), due_date (ISO 8601 string), last_update (ISO 8601 string), blockers_count (integer), description (string), tags (array of strings).
   - Purpose: the dashboard fetches this file and renders cards and summary metrics. Keep the data deterministic for tests.

3. Implement HTML skeleton + rendering (Coder) — app/index.html
   - Build a single-page, accessible HTML file that:
     - References app/styles.css.
     - Includes a header (title "Project Pulse"), a summary area (counts or aggregate metrics), and a container element with a deterministic class such as .dashboard and an id such as #projects-list where cards will render.
     - Adds a small inline script or a local script file that:
       - Fetches /project-data.json relative to the app directory.
       - Renders project cards into the #projects-list container using the Designer's CSS hooks (.project-card).
       - Shows a graceful error state (element with id #error-state) when fetch fails or JSON is malformed.
       - Uses semantic HTML and ARIA where appropriate (cards as role="listitem", container role="list").
     - Include minimal keyboard interactions (tab focusable cards; clicking/tapping toggles a short detail expansion).

4. Create baseline styling (Designer) — app/styles.css
   - Provide the Designer-created CSS file at app/styles.css implementing:
     - .dashboard (grid layout, responsive breakpoints: 1 column mobile / 2 tablet / 3 desktop).
     - .project-card (card layout, padding, accessible color usage).
     - Status badge classes (e.g., .status-on-track, .status-at-risk, .status-blocked) with accessible colors.
     - Progress bar styling (visual and aria-valuenow).
     - Focus outlines for keyboard users and readable typography.
   - Add CSS variables at :root for color tokens so the Coder/Orchestrator can easily change branding.

5. Add deterministic launch configuration (Coder) — .vscode/launch.json
   - Create a launch configuration that opens the dashboard URL in a browser:
     - Configuration name: "Launch Project Pulse".
     - Type: "pwa-chrome" (or "pwa-msedge" if preferred); request: "launch".
     - URL: http://127.0.0.1:5000/index.html
     - webRoot: ${workspaceFolder}/app
     - preLaunchTask: serve-app (matches the tasks.json task).
   - Rationale: this is deterministic and portable across Codespaces. It opens the client-side browser to the forwarded port 5000; the server is started via tasks.json automatically.

6. Add automated server task (Coder) — .vscode/tasks.json
   - Implement a background shell task named "serve-app":
     - Command: python3 -m http.server 5000
     - Working directory: ${workspaceFolder}/app
     - Mark it isBackground so a preLaunchTask in launch.json can be used.
   - This ensures a single Run & Debug action starts the server and opens the dashboard.

7. Designer polish & accessibility pass (Designer)
   - Review the rendered cards and summary; refine spacing, color contrast, focus states, and responsive breakpoints.
   - Provide CSS comments identifying the hooks (.dashboard, .project-card, .status-*) to avoid accidental renames.

8. Integration & verification (Coder + Designer)
   - Run the manual verification steps below and fix any alignment, interaction, or accessibility issues.
   - Verify the preLaunchTask / launch flow works as expected.

File assignments (who owns what — explicit)
- app/project-data.json — Coder
  - Responsible for creating the deterministic JSON with specified schema.
- app/index.html — Coder
  - Responsible for HTML structure, minimal inline JS that fetches JSON and renders cards, server-friendly file layout.
- app/styles.css — Designer
  - Responsible for the full visual design, CSS variables, and implementing .dashboard, .project-card, and status classes.
- .vscode/launch.json — Coder
  - Responsible for the deterministic launch configuration that opens http://127.0.0.1:5000/index.html.
- .vscode/tasks.json — Coder
  - Responsible for the automated background task that serves the app on port 5000.

Explicit responsibilities by role
- Designer
  - Create responsive, accessible CSS in app/styles.css using the hooks .dashboard, .project-card, .status-on-track/.status-at-risk/.status-blocked and document these hooks via CSS comments.
  - Provide color tokens, spacing scale, and any icon or small SVG assets used by the cards.
  - Do accessibility pass (contrast, keyboard focus, readable font sizes).
- Coder
  - Create app/index.html and wire up a deterministic minimal renderer that fetches app/project-data.json.
  - Create app/project-data.json with deterministic sample data.
  - Create .vscode/launch.json and .vscode/tasks.json so a reviewer can launch the app deterministically via Run & Debug.
  - Implement minimal error handling and one keyboard interaction (card expand/toggle).
  - Keep code dependency-free (no npm, no bundler). Use only standard browser APIs and a static Python server for preview.

Dependencies (internal and external)
- Internal:
  - app/index.html depends on app/styles.css and app/project-data.json.
  - The launch configuration depends on the reviewer starting a server at port 5000 via tasks.json.
- External / environment:
  - Codespace / development machine must have python3 installed for the static server (python3 -m http.server 5000).
  - A modern browser on the reviewer/client machine (Chrome / Edge / Firefox); launch.json uses pwa-chrome but will also work with pwa-msedge if preferred.
  - VS Code remote Codespace environment with the ability to forward port 5000 to the client machine (Codespaces does this by default).
- No additional npm or package dependencies are required.

Which work can run in parallel and which must be sequential
- Can run in parallel:
  - Coder creates app/project-data.json and the HTML skeleton (structure and minimal JS) while Designer simultaneously builds app/styles.css. Justification: the HTML structure and data schema are stable and the Designer's CSS only needs to match the agreed class hooks. This reduces friction; merge conflicts are unlikely because files are separate.
  - Coder can also create .vscode/launch.json and .vscode/tasks.json in parallel with the above.
- Must run sequentially or in short-sync loops:
  - Final integration testing (step 8) must be sequential: after both CSS and HTML/JS are ready, the team integrates and iterates.
  - Accessibility polish and any DOM-class renames must happen after an initial skeleton is rendered so Designer can tune styles against real content.
- Rationale: separation of concerns (data & structure vs. presentation) allows parallel work with minimal overlap; final integration is necessary to verify visual/interaction correctness.

Edge cases to handle
- Fetch failure / server not running:
  - Show a human-friendly error in the page telling the reviewer to start the server (with exact command: python3 -m http.server 5000 from app/), or open app/index.html via the launch config.
- Running via file:// (no server):
  - Modern browsers block fetch() from file:// to local files. Make clear in README and in-page message that a server must be used.
- Empty project-data.json (empty array):
  - Render an empty-state UI with a clear message: "No active projects found."
- Malformed JSON or missing keys:
  - Show an error state with the raw error logged to the console and a simplified message in UI.
- Long strings or many tags:
  - Truncate or wrap long names; show ellipses with full value available in the expanded card or via title attribute.
- Large data sets:
  - Keep preview limited to 3–6 projects; if future extension, implement lazy rendering or pagination.
- Color / contrast issues:
  - Designer must ensure AA contrast; status colors must be distinct for color-blind users (also use icons or patterns if possible).

Validation expectations (how to verify locally in the Codespace)
1. Start the preview server (automated via tasks.json):
   - From Run & Debug, run the "Launch Project Pulse" configuration which will start the serve-app task and open the dashboard.
   - Manual fallback: open a Codespace terminal and run:
     - cd app
     - python3 -m http.server 5000
2. Open the dashboard:
   - Expected URL to open: http://127.0.0.1:5000/index.html
   - From VS Code: Run the "Launch Project Pulse" configuration in the Run & Debug panel (it should open the above URL in the reviewer's browser).
3. Acceptance criteria — visual checks:
   - Header displays "Project Pulse".
   - The page shows 3–6 project cards (matches number of items in app/project-data.json).
   - Cards show: project name, owner, status badge (green/amber/red mapping to on-track/at-risk/blocked), progress bar with a percentage matching the JSON progress value, and due date.
   - The layout is responsive:
     - Desktop: 3 columns
     - Tablet: 2 columns
     - Mobile: 1 column
   - Focus states are visible when tabbing through cards.
4. Acceptance criteria — functional checks:
   - The summary area shows counts (e.g., number of blocked projects) consistent with JSON data.
   - Clicking or pressing Enter on a card toggles a detail view or expands to show description and last_update.
   - If the server is not running, the page displays a clear error message instructing how to start the server.
   - Console contains no unhandled exceptions during JSON fetch and rendering.
5. Minimal performance:
   - The static preview loads and renders within a second for the small dataset.

Work that can run in parallel (summary)
- Coder: project-data.json + index.html skeleton + launch.json + tasks.json
- Designer: styles.css
- These can be done concurrently. Integration and accessibility pass must follow.

Work that must be sequential (summary)
- Integration & polish after both skeleton and CSS exist.
- Accessibility fixes after integration.

Validation deliverables to produce when done
- A functioning Codespace preview where a reviewer can:
  - Start server via Run & Debug and open http://127.0.0.1:5000/index.html
  - See correctly styled dashboard with data from app/project-data.json and pass acceptance checks above.

Open questions for the Orchestrator
1. Port choice: Is port 5000 acceptable? (Default in this plan; can change if required.)
2. Preference for the browser in launch.json: Chrome (pwa-chrome) or Edge (pwa-msedge)? Either is acceptable; which is preferred for your reviewers?
3. Data fidelity: Should app/project-data.json be realistic production-like sample data, or minimal demo data? (Plan assumes deterministic small demo dataset.)
4. Additional feature scope: Should cards link to a detail page/modal, or is a simple inline expansion sufficient for the first preview?
5. Branding: Any color tokens, font preferences, or Mona brand assets to include in the CSS?

Notes, assumptions, and risks
- Assumption: Codespace has python3 available; running python3 -m http.server 5000 will serve the app directory.
- Risk: If the reviewer's machine cannot open the forwarded Codespace port automatically via launch.json (e.g., corporate firewall), they must use the Codespaces port preview or copy the forwarded URL from the Ports panel.
- To reduce risk: include clear instructions printed in the top of app/index.html (or a small README) telling the reviewer how to start the server and the exact URL to open.

Deliverable checklist (what the Orchestrator should expect to find after tasks are completed)
- docs/project-pulse-plan.md (this plan)
- app/index.html (Coder)
- app/styles.css (Designer)
- app/project-data.json (Coder)
- .vscode/launch.json (Coder)
- .vscode/tasks.json (Coder)

If you want, I can:
- Produce the step-by-step issue cards for the Orchestrator to assign to Coder and Designer (one file per task).
- Produce a minimal checklist-style PR description to use when the team makes the changes.

Please confirm:
- port (5000 ok?) — confirmed
- automated task (.vscode/tasks.json) included — confirmed
