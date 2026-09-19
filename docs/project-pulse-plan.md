# Project Pulse — Implementation Plan

## 1. Overview

Project Pulse is a **static, local dashboard** built with plain HTML, CSS, and JavaScript (no build step, no framework, no backend). It displays a grid of project cards, each showing:

- Project name
- **Status** (`On Track`, `At Risk`, `Delayed`)
- **Priority** (`High`, `Medium`, `Low`)
- Supporting metadata (description, owner, due date)

Data is sourced from a local JSON file (`app/project-data.json`) and rendered into the DOM at runtime via a small `fetch`-based script. The dashboard must run entirely from static files, previewable in VS Code via a launch/preview configuration, with no external dependencies or network calls beyond loading the local JSON.

The `app/` directory does not exist yet — all files below are net-new.

## 2. File Assignments

| File | Owner | Notes |
|---|---|---|
| `app/index.html` | **Split ownership** — Designer (structure pass) then Coder (scripting pass) | Designer defines semantic HTML skeleton: `<header>`, a `.dashboard` container (e.g. `<main class="dashboard">`), an accessible page title, and a template/contract for `.project-card` markup (either a static example card or a `<template>` element). Coder adds the `<script>` tag(s) that load data and populate the dashboard, plus any container elements needed purely for JS mounting (e.g., an empty state container) — coordinated with Designer, not overwriting Designer's structural choices. |
| `app/styles.css` | **Designer** (full ownership) | All visual design: layout, spacing, typography, color system for status badges and priority indicators, responsive breakpoints. Must style against the `.dashboard` and `.project-card` hooks (and any sub-hooks Designer defines, e.g. `.status-badge`, `.priority-high`) so Coder's generated markup renders correctly without needing inline styles. |
| `app/project-data.json` | **Coder** (full ownership) | Sample/mock data array. Each entry: `id`, `name`, `status`, `priority`, `description`, `owner`, `dueDate` (and any additional fields Coder deems useful, e.g. `tags`). Coder defines and documents this schema early (see Dependencies). |
| `.vscode/launch.json` | **Coder** (full ownership) | Config to preview/serve `app/index.html` locally (e.g., Live Preview extension config, or a simple static-server launch/task pairing). Must reference the correct relative path to `app/index.html`. Repo already has `.vscode/tasks.json` for an unrelated Copilot CLI task — Coder should add `launch.json` alongside it without modifying `tasks.json`. |

Optional (recommended, see §6): `app/app.js` — if the team chooses to separate scripting from `index.html`, this file is **Coder**-owned exclusively, referenced from `index.html` via a single `<script src="app.js" defer></script>` tag that Designer leaves as a placeholder.

## 3. Designer Responsibilities

- **Information architecture:** Decide dashboard layout — header/title area, optional filter/summary region (if in scope, see Open Questions), main grid/list of project cards.
- **Semantic & accessible markup:**
  - Use proper heading hierarchy (`<h1>` for dashboard title, `<h2>`/`<h3>` per card or section as appropriate).
  - Use landmark elements (`<header>`, `<main>`, `<section>`) so assistive tech can navigate.
  - Ensure status/priority indicators are not conveyed by color alone — include visually-hidden text or `aria-label`s (e.g., `<span class="status-badge status-on-track"><span class="sr-only">Status:</span> On Track</span>`).
  - Define the exact markup shape of a `.project-card` (via a static sample card in the HTML or a `<template>` element) so Coder can clone/populate it predictably.
- **Visual design system:**
  - Distinct, accessible-contrast color treatment per status (On Track / At Risk / Delayed) and per priority (High / Medium / Low).
  - Card layout: spacing, elevation/border, typography hierarchy for name vs. metadata.
- **Responsive behavior:** Grid should reflow from multi-column (desktop) to single-column (mobile); define breakpoints in `styles.css`.
- **CSS hooks contract (binding for Coder):**
  - `.dashboard` — the grid/list container Coder will insert cards into.
  - `.project-card` — the repeatable card unit Coder will render per project.
  - Any additional hooks Designer introduces (e.g., `.status-badge`, `.priority-badge`, `.empty-state`) **must be documented in code comments in `index.html` or `styles.css`** so Coder can rely on them without guessing.

## 4. Coder Responsibilities

- **Data layer (`app/project-data.json`):** Define and populate a sample dataset (suggest 6–10 sample projects covering all status/priority combinations for good test coverage).
- **Rendering logic (script in `index.html` or separate `app.js`):**
  - Fetch `project-data.json` via `fetch()` (works with a local static server; note `file://` fetch will fail in some browsers — this is a reason to require the `.vscode/launch.json` preview server rather than opening the HTML file directly).
  - Parse JSON and validate shape defensively (each project has required fields before rendering).
  - For each project, clone/populate the Designer's `.project-card` template and insert into `.dashboard`.
  - Map `status` and `priority` values to the CSS classes Designer defined (e.g., `status-${slugify(status)}`).
- **Error handling:**
  - Fetch failure (network/file error) → show a visible, accessible error message in place of the dashboard grid, not a blank page or console-only error.
  - Malformed/empty JSON (empty array, missing fields) → render an "empty state" message using Designer's `.empty-state` hook if provided, or a sensible fallback.
  - Guard against missing optional fields (e.g., no `dueDate`) without throwing.
- **Preview/run config (`.vscode/launch.json`):** Provide a working local preview configuration (e.g., Live Preview extension launch config or equivalent) that serves the `app/` directory so `fetch()` works correctly, targeting `app/index.html`.

## 5. Dependencies

1. **Schema before rendering logic:** Coder must define the `project-data.json` field shape (id/name/status/priority/description/owner/dueDate) before finalizing the rendering JS, since the JS maps directly to these fields.
2. **HTML skeleton + CSS hooks before JS wiring:** Designer's `.dashboard` container and `.project-card` template/markup shape must exist before Coder can write DOM-insertion logic — Coder needs to know exactly what to clone/populate.
3. **`index.html` existing at a known path before `launch.json`:** Coder cannot finalize the preview config until `app/index.html` exists at its final path (`app/index.html`).
4. **Status/priority naming alignment:** The literal strings used in `project-data.json` (e.g., `"On Track"`, `"At Risk"`, `"Delayed"`) must match the class-name mapping logic Coder writes and the CSS selectors Designer authors — this should be agreed as a small shared contract (e.g., a comment block listing exact enum values) before either side finalizes.

## 6. Parallel Work Decisions

**Can run in parallel (no file/data conflict):**
- Designer working on `app/styles.css` (visual design system, responsive rules) — independent of data shape.
- Coder drafting `app/project-data.json` sample data and schema — independent of visual styling.
- Coder drafting the `.vscode/launch.json` config skeleton, since it only needs to know the eventual path `app/index.html` will live at (can be stubbed/assumed early).

**Must run sequentially / coordinated:**
- `app/index.html` is touched by both Designer and Coder. To avoid merge conflicts and ambiguity:
  - **Recommended approach:** Designer completes the structural pass first (semantic skeleton, `.dashboard` container, sample `.project-card` markup/template, placeholder `<script src="app.js" defer></script>` tag). Only after that is Coder allowed to add `app/app.js` — a **separate, Coder-owned file** — so Coder never needs to re-edit `index.html`'s body content, only leaves the single script tag Designer already placed.
  - If a separate `app.js` is not desired, fall back to strict sequencing: Designer finishes `index.html` structure → hands off → Coder adds `<script>` block, with no further Designer edits after handoff.
- `.vscode/launch.json` should be finalized only after `app/index.html`'s path is confirmed (trivial dependency, but sequence-wise it comes after the HTML file is created, even if just as a stub).

## 7. Validation Expectations

- Launching the preview via `.vscode/launch.json` opens `app/index.html` in a browser/simple-browser view with **no console errors**.
- All sample projects defined in `app/project-data.json` render as visible `.project-card` elements inside `.dashboard`.
- Status and priority are **visually distinguishable** (color + text, not color alone) for every status/priority combination present in the sample data.
- Simulate an empty/missing `project-data.json` (e.g., temporarily rename or empty the array) and confirm the dashboard shows a graceful empty-state message instead of crashing or showing a blank screen.
- Resize the browser/preview to common breakpoints (mobile ~375px, tablet ~768px, desktop ~1280px) and confirm the `.dashboard` grid reflows sensibly with no overflow or unreadable text.
- Accessibility spot-check: heading hierarchy is logical, status/priority badges have accessible text (not icon/color-only), and there are no obvious keyboard-navigation traps.

## 8. Open Questions

- Exact number and content of sample projects in `app/project-data.json` — plan assumes 6–10 covering all status × priority combinations; confirm with Orchestrator/stakeholder if a specific count or real project names are expected.
- Should rendering JS live inline in `app/index.html` or in a separate `app/app.js`? This plan recommends the separate-file approach to reduce contention on `index.html`, but Orchestrator should confirm which pattern the exercise expects.
- Is filtering/sorting (e.g., filter by status or priority) in scope for this iteration, or purely a static read-only display? This affects whether Designer needs to design filter controls and whether Coder needs additional interactive logic.
- Exact mechanism for `.vscode/launch.json` — Live Preview extension config vs. a lightweight static server task — not yet confirmed; Coder should pick based on what's available in the devcontainer and document the choice.
- Should an "empty state" or "error state" have dedicated Designer-authored markup/CSS (e.g., `.empty-state` class), or should Coder generate that state's markup dynamically with inline fallback styling? Recommend Designer provide the hook to keep styling consistent.
