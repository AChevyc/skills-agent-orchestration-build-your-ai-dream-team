# Mona's Project Pulse Dashboard — Implementation Plan

## 1. Summary

Mona's team needs a small, static Project Pulse dashboard that contributors can
open locally to quickly see which projects are active, who owns them, their
current status, recent activity, and priority. The deliverable is a static web
app under `app/` plus a VS Code launch configuration under `.vscode/`.

Per the brief in `.github/project-pulse-brief.md` and the team conventions in
`docs/agent-team.md`:

- The dashboard is a static HTML/CSS app driven by a local JSON data file.
- It must look like a polished dashboard (project cards, status badges,
  priority treatment, readable spacing) — not a bare HTML page.
- A VS Code launch configuration named **"Run Project Pulse Dashboard"** must
  serve the app from the `app/` directory and open `index.html` (not a
  directory listing).
- Work is split between two specialist agents:
  - **Designer** — UI/UX, visual design, layout, accessibility, information
    hierarchy. Owns `app/styles.css` and the structural/visual markup in
    `app/index.html`.
  - **Coder** — data structures, JS logic (if any), and support configuration.
    Owns `app/project-data.json` and `.vscode/launch.json`, and contributes
    any data-loading / rendering script that lives inside `app/index.html`.

Both agents follow existing repository patterns: no committing/pushing (the
learner controls git via Copilot CLI), deterministic file layouts, and the
required CSS hooks (`.dashboard`, `.project-card`) and JSON shape (top-level
`projects` array) that the exercise validation depends on.

## 2. Ordered implementation steps

1. **Define the data contract (Coder).** Author `app/project-data.json` with a
   top-level `projects` array. Each project object includes: `name`, `owner`,
   `status`, `recentActivity`, `priority`. This step establishes the shape
   every other file depends on.
2. **Draft the HTML skeleton and rendering logic (Coder + Designer, coordinated
   on a single file).** Create `app/index.html` with:
   - Semantic document structure (`<header>`, `<main>`, dashboard container
     with class `dashboard`, and a container for project cards using the
     `project-card` class).
   - A small inline `<script>` (Coder) that fetches `project-data.json` and
     renders one `.project-card` per project, exposing `name`, `owner`,
     `status`, `recentActivity`, and `priority`.
   - A `<link rel="stylesheet" href="styles.css">` reference.
   - Accessible landmarks, headings, and ARIA where needed (Designer input on
     structure).
3. **Style the dashboard (Designer).** Author `app/styles.css` targeting the
   deterministic hooks (`.dashboard`, `.project-card`, plus status/priority
   badge classes agreed with the Coder in step 2). Include polished visual
   affordances required by the brief and validation: `border-radius`,
   `box-shadow`, spacing, typography, color contrast, responsive layout.
4. **Add the launch configuration (Coder).** Create `.vscode/launch.json` as
   strict JSON (no comments) with a configuration named **"Run Project Pulse
   Dashboard"** that:
   - Serves from `${workspaceFolder}/app` (i.e. `cwd` = `${workspaceFolder}/app`).
   - Starts a simple local static server (e.g. `python3 -m http.server`).
   - Opens `http://localhost:<port>/index.html` in the browser (via
     `serverReadyAction` or an equivalent deterministic mechanism) so learners
     see the dashboard, not a directory listing.
5. **Cross-check + validate (both agents).** Open the launch config, confirm
   the dashboard renders real data from `project-data.json`, confirm styling
   hooks are present, and run `bash scripts/validate-exercise.sh` if the
   learner wants to sanity-check the template invariants.

## 3. File assignments

| File | Responsible agent | What that agent must do |
|---|---|---|
| `app/index.html` | **Designer (primary for structure/markup) + Coder (data-loading script)** | Designer defines the semantic structure, dashboard container (`class="dashboard"`), project card template markup (`class="project-card"` and any badge classes for `status` / `priority`), headings, landmarks, and accessible labels; links `styles.css`. Coder adds the inline `<script>` that fetches `project-data.json` and renders one card per project, populating `name`, `owner`, `status`, `recentActivity`, `priority`. Because the file has two authors, the Orchestrator MUST sequence them (Designer first, then Coder appends the script) to avoid edit conflicts. |
| `app/styles.css` | **Designer** | Provide a polished visual design targeting `.dashboard` and `.project-card` at minimum, plus status and priority treatments. Must include `border-radius` and `box-shadow` (required for validation), readable typography, spacing, color contrast that meets WCAG AA, and a responsive layout (grid or flex that reflows on narrow screens). |
| `app/project-data.json` | **Coder** | Strict JSON with a top-level `projects` array. Each project object contains `name`, `owner`, `status`, `recentActivity`, `priority`. Provide several realistic sample projects (recommend 4–6) with a variety of statuses and priorities so the dashboard visibly exercises the badge styles. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. One configuration named exactly **"Run Project Pulse Dashboard"**. `cwd` set to `${workspaceFolder}/app`. Serves the static app (e.g. `python3 -m http.server`) and opens `http://localhost:<port>/index.html` on server ready. Deterministic port, name, cwd, and URL. |

## 4. Designer responsibilities (detailed)

- **Information architecture**
  - Page header with the "Project Pulse" title and a short contributor-friendly
    subtitle.
  - Main region containing the `.dashboard` grid of `.project-card` elements.
  - Each card surfaces, in a clear hierarchy: project `name` (heading), `owner`
    (secondary), `status` (badge), `priority` (badge or accent), `recentActivity`
    (supporting text).
- **Visual design**
  - Polished, card-based layout — not a raw HTML page.
  - Use `border-radius` and `box-shadow` on cards (required by validation and
    the brief).
  - Distinct color treatments for status values (e.g. active / at-risk /
    blocked / shipped) and priority levels (e.g. low / medium / high). Agree
    on class names with Coder before step 2 so the script emits matching
    classes.
  - Readable typography scale, generous spacing, and consistent alignment.
- **Responsive behavior**
  - Grid that reflows to a single column on narrow viewports.
  - No horizontal scrolling at common widths (≥ 320px).
- **Accessibility**
  - Semantic landmarks (`<header>`, `<main>`), a single `<h1>`, and `<h2>`/`<h3>`
    for card titles.
  - Sufficient color contrast (WCAG AA) for text and badge text.
  - Do not rely on color alone to communicate status/priority — pair with text
    labels.
  - Focus states for any interactive element added later.
- **Deliverable hooks (deterministic, non-negotiable)**
  - `.dashboard` selector present in `styles.css` and applied in `index.html`.
  - `.project-card` selector present in `styles.css` and applied in
    `index.html`.
  - `border-radius` and `box-shadow` declarations present in `styles.css`.

## 5. Coder responsibilities (detailed)

- **`app/project-data.json`**
  - Strict JSON, UTF-8, no trailing commas, no comments.
  - Top-level object with a single `projects` key whose value is an array.
  - Each project object has exactly (at minimum) `name`, `owner`, `status`,
    `recentActivity`, `priority` — all strings.
  - Provide 4–6 sample projects with a variety of `status` and `priority`
    values to visibly exercise the design.
- **Rendering script inside `app/index.html`**
  - Use `fetch('project-data.json')` to load data at runtime.
  - Render one `.project-card` per project using the class names agreed with
    Designer (including status/priority modifier classes).
  - Escape/encode text content when injecting into the DOM (use
    `textContent`, not `innerHTML`, for user-visible strings).
  - Handle fetch failure gracefully: render a visible, styled error message
    inside the dashboard container instead of leaving the page blank.
  - Handle empty `projects` arrays with an "No projects yet" empty state.
- **`.vscode/launch.json`**
  - Strict JSON, no comments.
  - Single configuration named exactly **"Run Project Pulse Dashboard"**.
  - `cwd`: `${workspaceFolder}/app`.
  - Runs a deterministic static server (recommended: `python3 -m http.server`
    on a fixed port such as `8000`).
  - Uses `serverReadyAction` (or equivalent) with a pattern matching the
    server's "Serving HTTP" line, action `openExternally`, and
    `uriFormat: "http://localhost:%s/index.html"` — so the browser opens
    the dashboard, not a directory listing.
  - Do not include workspace-invalid fields; validate that
    `python3 -m json.tool .vscode/launch.json` succeeds.

## 6. Dependencies between steps/files

1. `app/project-data.json` (step 1) → **must exist before** the rendering
   script in `app/index.html` (step 2) is finalized. The script's field
   references depend on the JSON shape.
2. `app/index.html` structural markup (step 2, Designer portion) → **must
   exist before** `app/styles.css` (step 3) can be finalized, because CSS
   selectors target the concrete class names (`.dashboard`, `.project-card`,
   status/priority classes) present in the markup.
3. Designer's status/priority class-naming decisions (step 2/3) → **must be
   agreed before** the Coder's rendering script emits classes, otherwise the
   badges won't style correctly.
4. `.vscode/launch.json` (step 4) → **must exist after** the final `app/`
   folder structure (specifically `app/index.html`) is confirmed, because the
   launch config's `cwd` and open URL reference `index.html` inside `app/`.
5. Validation (step 5) → depends on all four files being present.

## 7. Parallel work decisions

**Can run in parallel (no scope overlap, no data dependency):**

- `app/project-data.json` (Coder) can be authored **in parallel** with the
  Designer's initial visual exploration / mock of the dashboard, since the
  JSON schema is fixed by the brief.
- Once the class-name contract for `.dashboard`, `.project-card`, and
  status/priority modifiers is agreed, the Designer can finalize
  `app/styles.css` **in parallel** with the Coder authoring `.vscode/launch.json`
  — the two files never touch each other.

**Must run sequentially (shared file or data dependency):**

- `app/index.html` is a **shared-ownership file**. Designer writes the
  structural markup first, then Coder appends the inline data-loading script.
  These two edits must not happen concurrently.
- The Designer's `app/styles.css` cannot be finalized until the concrete class
  names (from the agreed contract + the actual `index.html` markup) are locked.
- `.vscode/launch.json` cannot be validated end-to-end until `app/index.html`
  exists at the path the launch URL points to.

**Recommended orchestration:**

- Phase A (parallel): Coder writes `project-data.json`; Designer proposes
  markup skeleton + class-name contract for `index.html`.
- Phase B (sequential on `index.html`): Designer commits markup; Coder appends
  script.
- Phase C (parallel): Designer writes `styles.css`; Coder writes
  `.vscode/launch.json`.
- Phase D (sequential): Both agents validate.

## 8. Edge cases to handle

- **Empty `projects` array** — the dashboard should render an accessible empty
  state ("No projects yet"), not a blank page.
- **Missing/optional field on a project object** — the script should render a
  neutral placeholder (e.g. an em dash) rather than the string `undefined`.
- **Fetch failure** (e.g. file opened via `file://` where `fetch` is blocked,
  or JSON parse error) — render a styled inline error message inside
  `.dashboard`. This is why the launch configuration serves the app over HTTP
  instead of relying on `file://`.
- **Unknown status or priority values** — CSS should fall back to a neutral
  badge style if the modifier class isn't recognized.
- **Long project names, owners, or activity strings** — wrap gracefully; do
  not break the card grid.
- **Narrow viewports (≤ 480px)** — cards must reflow to a single column with
  no horizontal scroll.
- **Reduced color vision / high-contrast mode** — status/priority meaning
  must not rely on color alone; always pair with text.
- **Port already in use** for the launch server — document the port choice
  (e.g. 8000) so learners can spot the collision; deterministic port keeps
  the launch config reproducible.
- **JSON strictness** — no trailing commas or comments in either
  `project-data.json` or `.vscode/launch.json`; both must pass
  `python3 -m json.tool`.
- **XSS in rendered strings** — always use `textContent` when injecting
  values from JSON into the DOM.

## 9. Validation expectations

- **File presence**: `app/index.html`, `app/styles.css`, `app/project-data.json`,
  and `.vscode/launch.json` all exist.
- **JSON validity**:
  - `python3 -m json.tool app/project-data.json` succeeds.
  - `python3 -m json.tool .vscode/launch.json` succeeds.
- **Data contract**: `app/project-data.json` has a top-level `projects` array
  whose elements each contain `name`, `owner`, `status`, `recentActivity`,
  `priority`.
- **Markup hooks**: `app/index.html` contains `dashboard` and `project-card`
  class usages, and references `styles.css` and `project-data.json`.
- **Style hooks**: `app/styles.css` contains `.dashboard`, `.project-card`,
  `border-radius`, and `box-shadow` declarations.
- **Launch configuration**:
  - Configuration named exactly **"Run Project Pulse Dashboard"**.
  - `cwd` = `${workspaceFolder}/app`.
  - Opens `http://localhost:<port>/index.html` on server ready (not a
    directory listing).
- **Runtime behavior** (manual):
  - Launch from VS Code → server starts, browser opens directly to the
    dashboard.
  - All sample projects render as cards with visible status/priority badges,
    owner, and recent activity.
  - Simulate a fetch failure (temporarily rename `project-data.json`) → the
    error message appears inside `.dashboard` instead of a blank page.
- **Accessibility (manual)**:
  - Single `<h1>`, semantic landmarks, sufficient color contrast, meaning not
    conveyed by color alone.
  - Keyboard focus visible on any interactive element.
- **Responsiveness (manual)**: Cards reflow at 320px, 768px, and 1280px
  without horizontal scroll.
- **Template invariants**: `bash scripts/validate-exercise.sh` continues to
  pass (learner answer files must not be committed to the template branch —
  the learner controls when/how to commit).

## 10. Open questions

1. **Server choice for the launch config** — is `python3 -m http.server`
   acceptable, or does Mona prefer a Node-based server (e.g. `npx serve`)?
   The brief does not specify. Recommend `python3 -m http.server` because it
   is preinstalled in the devcontainer and matches the deterministic style
   used elsewhere in the exercise.
2. **Fixed port** — port `8000` is proposed for determinism; is that
   acceptable, or should the launch config allow VS Code to auto-assign?
3. **Additional project fields** — the brief lists `name`, `owner`, `status`,
   `recentActivity`, `priority`, plus a "short contributor-friendly summary".
   Should `summary` be added as a sixth field on each project object, or is
   the header-level tagline sufficient? Recommend adding an optional
   `summary` string per project and rendering it inside the card.
4. **Status and priority vocabularies** — are the allowed values enumerated
   anywhere? Recommend Coder + Designer agree on a small set
   (status: `active`, `at-risk`, `blocked`, `shipped`; priority: `low`,
   `medium`, `high`) and document the mapping in the JSON sample data plus
   the CSS modifier classes.
5. **Filtering/sorting UI** — out of scope for v1? The brief asks only to
   "display" project status, so recommend deferring interactive controls.
6. **Dark mode** — not mentioned in the brief; recommend deferring but
   picking a palette that would extend cleanly to a `prefers-color-scheme`
   variant later.
