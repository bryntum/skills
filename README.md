# Bryntum Skills

Agent skills for working with [Bryntum](https://bryntum.com) components — Scheduler (incl. the former Scheduler Pro), Gantt, Calendar, Grid, and TaskBoard. Compatible with Claude Code and Codex.

The skills serve both 8.0 and 7.x apps from this one `main` branch: each skill that has 7.x differences starts with a version check that reads the app's `package.json` and loads its `references/v7.md` for a 7.x app; the others apply to both, with their few 8.0-only sentences marked "On 8.x". On 8.0 every package is `@bryntum/<product>-<tier>`; the Community tier needs no license key, while Business and Enterprise run as a 14-day trial until a key is set.

## Skills

### `bryntum` _(required)_

The core skill. Covers the version check, workspace and product identification, tiers and the license key, package installation (`@bryntum/<product>-<tier>`) and CSS styling, docs lookup via MCP, data loading rules, product-specific component defaults, sizing, and output verification. Always install this one — the framework and CRUD skills build on it.

### `bryntum-react`

React-specific patterns: `<BryntumXxx>` component from `@bryntum/<product>-react-<tier>`, Core widgets from `@bryntum/core-react-<tier>`, JSX config props, `useRef` for instance access, and StrictMode double-mount handling.

### `bryntum-angular`

Angular-specific patterns: `<bryntum-xxx>` selector, `[prop]="..."` template binding, standalone component setup with `BryntumCoreModule` next to the product module, and how to separate Bryntum config from Angular's own `app.config.ts`.

### `bryntum-vue`

Vue patterns (`@bryntum/<product>-vue-<tier>`, no `-vue-3`): `<bryntum-xxx>` component, `v-bind` config spreading, and typing config objects as `BryntumXxxProps` for type-safe binding.

### `bryntum-vanilla`

Vanilla JS patterns: direct class import from `@bryntum/<product>-<tier>` (Core helpers from `@bryntum/core-<tier>`) and instantiation with `appendTo`.

### `bryntum-crud`

Backend persistence patterns: the project as CrudManager (`loadUrl`/`syncUrl`; Scheduler, Gantt, Calendar, TaskBoard) and AjaxStore (Grid), phantom ID handling, partial sync, and common data gotchas.

### `bryntum-drag-and-drop`

How to implement drag and drop from a grid onto a Scheduler or Gantt.

### `bryntum-theming`

Theme catalog (family sheets and light/dark variants), choosing a theme to match the host app (Material / Fluent), and light/dark switching via `color-scheme`, `DomHelper.colorScheme` and `DomHelper.setTheme()`.

### `bryntum-styling`

Custom event bar styling for Scheduler (`eventRenderer` layouts, event bar DOM structure, padding CSS variables, sticky content) and widget `rendition` variants for buttons, text fields and tooltips, plus the form look. Depth lives in `references/`.

### `bryntum-editor`

Customizing the built-in event/task editor — which editor each tier uses (`eventEdit`, `taskEdit`, Gantt Community's `basicTaskEdit`), adding/removing fields and tabs, and the supported way to swap in a fully custom dialog.

### `bryntum-features`

Concept-to-feature lookup: what a user asks for in plain language ("shade 11:00-13:00 as lunch", "grey out weekends", "a totals row", "export to Excel", "swimlanes", "critical path") mapped to the built-in feature that does it and the tier it needs, plus the commonly invented APIs that don't exist.

### `bryntum-from-extjs`

For codebases and habits coming from Ext JS. Maps Ext idioms to their Bryntum equivalents (`store.first()`, `component.down()`, `xtype`, `dataIndex`, `Ext.getCmp`, `Ext.Msg`) and flags the near-misses that fail silently rather than throwing.

### `bryntum-migrate`

Upgrading an existing Bryntum app to a newer version: detects installed and target versions, gathers the ordered upgrade guides / what's-new / changelogs across the product and the products it inherits from, cross-references them with your code, writes a migration plan you approve, then applies it (including the v6 → v7 CSS codemod and the 7.x → 8.0 tier package rename) and verifies the build.

---

## Installation

At minimum, install `bryntum` plus the skill for your framework.

### Easiest: the `skills` CLI

Run this in your project folder. The CLI detects your agents and lets you choose skills:

```bash
npx skills add bryntum/skills
```

### Manual install

Or copy the skills you need into your agent's skills folder with `degit`.

#### Claude Code

Skills live in `.claude/skills/` (project-local). Use `~/.claude/skills/` instead to install them globally for all projects.

**Install all skills at once:**

```bash
npx degit bryntum/skills/bryntum .claude/skills/bryntum
npx degit bryntum/skills/bryntum-react .claude/skills/bryntum-react
npx degit bryntum/skills/bryntum-angular .claude/skills/bryntum-angular
npx degit bryntum/skills/bryntum-vue .claude/skills/bryntum-vue
npx degit bryntum/skills/bryntum-vanilla .claude/skills/bryntum-vanilla
npx degit bryntum/skills/bryntum-crud .claude/skills/bryntum-crud
npx degit bryntum/skills/bryntum-drag-and-drop .claude/skills/bryntum-drag-and-drop
npx degit bryntum/skills/bryntum-theming .claude/skills/bryntum-theming
npx degit bryntum/skills/bryntum-styling .claude/skills/bryntum-styling
npx degit bryntum/skills/bryntum-editor .claude/skills/bryntum-editor
npx degit bryntum/skills/bryntum-features .claude/skills/bryntum-features
npx degit bryntum/skills/bryntum-from-extjs .claude/skills/bryntum-from-extjs
npx degit bryntum/skills/bryntum-migrate .claude/skills/bryntum-migrate
```

**Install individually** (e.g. React only):

```bash
npx degit bryntum/skills/bryntum .claude/skills/bryntum
npx degit bryntum/skills/bryntum-react .claude/skills/bryntum-react
```

#### Codex

Skills live in `.agents/skills/` (project-local). Use `~/.agents/skills/` instead to install them globally for all projects.

**Install all skills at once:**

```bash
npx degit bryntum/skills/bryntum .agents/skills/bryntum
npx degit bryntum/skills/bryntum-react .agents/skills/bryntum-react
npx degit bryntum/skills/bryntum-angular .agents/skills/bryntum-angular
npx degit bryntum/skills/bryntum-vue .agents/skills/bryntum-vue
npx degit bryntum/skills/bryntum-vanilla .agents/skills/bryntum-vanilla
npx degit bryntum/skills/bryntum-crud .agents/skills/bryntum-crud
npx degit bryntum/skills/bryntum-drag-and-drop .agents/skills/bryntum-drag-and-drop
npx degit bryntum/skills/bryntum-theming .agents/skills/bryntum-theming
npx degit bryntum/skills/bryntum-styling .agents/skills/bryntum-styling
npx degit bryntum/skills/bryntum-editor .agents/skills/bryntum-editor
npx degit bryntum/skills/bryntum-features .agents/skills/bryntum-features
npx degit bryntum/skills/bryntum-from-extjs .agents/skills/bryntum-from-extjs
npx degit bryntum/skills/bryntum-migrate .agents/skills/bryntum-migrate
```

**Install individually** (e.g. React only):

```bash
npx degit bryntum/skills/bryntum .agents/skills/bryntum
npx degit bryntum/skills/bryntum-react .agents/skills/bryntum-react
```

## Requirements

- [Claude Code](https://claude.ai/code) or [Codex](https://openai.com/codex/)
