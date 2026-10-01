# Bryntum Skills

Agent skills for working with [Bryntum](https://bryntum.com) components — Scheduler, Scheduler Pro, Gantt, Calendar, Grid, and TaskBoard. Compatible with Claude Code and Codex.

## Skills

### `bryntum` _(required)_

The core skill. Covers workspace and product identification, package installation and CSS styling, docs lookup via MCP, data loading rules, product-specific component defaults, sizing, and output verification. Always install this one — the framework and CRUD skills build on it.

### `bryntum-react`

React-specific patterns: `<BryntumXxx>` component, JSX config props, `useRef` for instance access, and StrictMode double-mount handling.

### `bryntum-angular`

Angular-specific patterns: `<bryntum-xxx>` selector, `[prop]="..."` template binding, standalone component setup, and how to separate Bryntum config from Angular's own `app.config.ts`.

### `bryntum-vue`

Vue 3-specific patterns: `<bryntum-xxx>` component, `v-bind` config spreading, and typing config objects as `BryntumXxxProps` for type-safe binding.

### `bryntum-vanilla`

Vanilla JS patterns: direct class import from `@bryntum/{product}` and instantiation with `appendTo`.

### `bryntum-crud`

Backend persistence patterns: CrudManager (Scheduler, Scheduler Pro, Gantt, Calendar, TaskBoard) and AjaxStore (Grid), phantom ID handling, partial sync, and common data gotchas.

### `bryntum-drag-and-drop`

How to implement drag and drop from a grid onto a Scheduler or Gantt.

### `bryntum-theming`

Theme catalog, choosing a theme to match the host app (Material / Fluent), and dynamic light/dark switching via `DomHelper.setTheme()`.

### `bryntum-styling`

Custom event bar styling for Scheduler / Scheduler Pro: `eventRenderer` layouts, event bar DOM structure, padding CSS variables, sticky content, and a worked two-line layout example.

### `bryntum-editor`

Customizing the built-in event/task editor — adding/removing fields and tabs via `eventEdit`/`taskEdit`, and the supported way to swap in a fully custom dialog.

### `bryntum-extjs-migration`

Migrating Ext JS apps to Bryntum 7 (vanilla JS, React, Angular or Vue): legacy Ext Scheduler / Gantt (`Sch.*`, `Gnt.*`), Bryntum inside an Ext shell, and plain Ext grids and tree grids. Covers Ext → Bryntum API mappings, data migration and verification.

---

## Installation

Copy the skills you need into your agent's skills folder. At minimum, install `bryntum` plus the skill for your framework.

### Claude Code

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
npx degit bryntum/skills/bryntum-extjs-migration .claude/skills/bryntum-extjs-migration
```

**Install individually** (e.g. React only):

```bash
npx degit bryntum/skills/bryntum .claude/skills/bryntum
npx degit bryntum/skills/bryntum-react .claude/skills/bryntum-react
```

### Codex

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
npx degit bryntum/skills/bryntum-extjs-migration .agents/skills/bryntum-extjs-migration
```

**Install individually** (e.g. React only):

```bash
npx degit bryntum/skills/bryntum .agents/skills/bryntum
npx degit bryntum/skills/bryntum-react .agents/skills/bryntum-react
```

## Requirements

- [Claude Code](https://claude.ai/code) or [Codex](https://openai.com/codex/)
