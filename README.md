# Bryntum Skills for Claude Code

This repository contains Claude Code skills for working with [Bryntum](https://bryntum.com) components — Scheduler, Scheduler Pro, Gantt, Calendar, Grid, and TaskBoard.

## Skills

### `bryntum` _(required)_

The core skill. Covers product identification, CSS setup, official demo mirroring, docs lookup via MCP, product-specific component defaults, sizing, dark mode, and data loading rules. Always install this one — the framework and CRUD skills build on it.

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

---

## Installation

Copy the skills you need into your `.claude/skills/` folder. At minimum, install `bryntum` plus the skill for your framework.

**Install all skills at once:**

```bash
npx degit bryntum/skills/bryntum ~/.claude/skills/bryntum
npx degit bryntum/skills/bryntum-react ~/.claude/skills/bryntum-react
npx degit bryntum/skills/bryntum-angular ~/.claude/skills/bryntum-angular
npx degit bryntum/skills/bryntum-vue ~/.claude/skills/bryntum-vue
npx degit bryntum/skills/bryntum-vanilla ~/.claude/skills/bryntum-vanilla
npx degit bryntum/skills/bryntum-crud ~/.claude/skills/bryntum-crud
```

**Install individually** (e.g. React only):

```bash
npx degit bryntum/skills/bryntum ~/.claude/skills/bryntum
npx degit bryntum/skills/bryntum-react ~/.claude/skills/bryntum-react
```

## Requirements

- [Claude Code](https://claude.ai/code)
