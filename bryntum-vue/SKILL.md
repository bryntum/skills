---
name: bryntum-vue
description: >
  Vue 3-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Scheduler Pro, Gantt, Calendar, Grid,
  TaskBoard) in a Vue 3 project, on the 8.0 tier packages (`@bryntum/<product>-vue-<tier>`) or on
  7.x (`@bryntum/*-vue-3`). Trigger when the project has Vue in package.json or when the user asks
  about Vue + Bryntum integration, v-bind, BryntumXxxProps, or @bryntum/*-vue packages.
metadata:
  tags: bryntum, vue, vue3, tier
---

## Version check — do this first

Read the app's `package.json` and pick the branch (in a monorepo, the workspace package that will import Bryntum). Every code block in this file is 8.0 code.

| `@bryntum/*` dependencies | Version | Do |
|---|---|---|
| Any tiered name: `@bryntum/<product>-community`, `-business` or `-enterprise` (untiered names next to it, other than `@bryntum/demo-resources`, are migration leftovers: finish with `bryntum-migrate`) | **8.0** | Follow this file |
| Only untiered names: `@bryntum/<product>`, a wrapper without a tier suffix, `-trial`, `-thin`, `-lib` or `@bryntum/schedulerpro` (ignore `@bryntum/demo-resources`) | **7.x or older** | Load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-vue/references/v7.md). Read the installed version from `node_modules/@bryntum/<product>/package.json` (`<product>-thin` for a thin app), or from the lockfile when nothing is installed; below 7.0, say this skill covers 7.x and 8.x and offer `bryntum-migrate`. Don't upgrade unless asked — that is `bryntum-migrate` |
| None, and no `build/` folder | **New app on 8.0** | Follow this file |
| No `@bryntum/*` dependency but a `build/` folder (distribution archive, see Workspace classification in the `bryntum` skill) | **from `build/package.json`** | `version` 8.x: follow this file. 7.x: load `references/v7.md` |

---

## Quick-start guide

8.x only. On 7.x, `references/v7.md` replaces this section.

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/<product>/docs-llm/guide/<Product>/quick-start/vue.md`

The page names the Enterprise packages; substitute the app's tier.

---

## Vue 3

### Packages and imports

8.x only. On 7.x, `references/v7.md` replaces this section.

Framework wrapper package: `@bryntum/<product>-vue-<tier>` — `vue` with no `-3` (the 7.x `-vue` package was the removed Vue 2 wrapper). It depends on `@bryntum/core-vue-<tier>`, the wrappers of the products below it and the product package, so those arrive with it.

A wrapper exports its own product's components and their `…Props` types; there is no plugin or global registration — import per component, from the package that owns it, and declare every package you import from in `package.json`:

```typescript
import { BryntumGantt, BryntumGanttProjectModel } from '@bryntum/gantt-vue-enterprise';
import type { BryntumGanttProps } from '@bryntum/gantt-vue-enterprise';
import { BryntumButton, BryntumCombo } from '@bryntum/core-vue-enterprise';   // Core widgets
import { BryntumGrid } from '@bryntum/grid-vue-enterprise';                   // another product on the page
import { StringHelper } from '@bryntum/core-enterprise';                       // Core helpers
import type { TaskModel } from '@bryntum/gantt-enterprise';                    // product classes and types
```

Scheduler Pro is Scheduler at the Enterprise tier: `BryntumScheduler` and `BryntumSchedulerProjectModel` from `@bryntum/scheduler-vue-enterprise`. `BryntumSchedulerPro` still exists but wraps the deprecated `SchedulerPro` class, which logs a deprecation — use `BryntumScheduler`.

A Grid feature the product does not load itself (`cellTooltip`, `quickFind`) registers only after `import '@bryntum/grid-<tier>';` — see the `bryntum` skill, Imports.

### Component

Spread config with `v-bind`:

```vue
<bryntum-gantt v-bind="ganttConfig" />
```

Or bind individual props:

```vue
<bryntum-gantt :columns="columns" :project="project" />
```

### Type-safe v-bind

8.x only. On 7.x, `references/v7.md` replaces this section.

Type the config object as `BryntumGanttProps` so `v-bind` type-checks:

```typescript
import type { BryntumGanttProps } from '@bryntum/gantt-vue-enterprise';

const ganttConfig: BryntumGanttProps = {
    viewPreset : 'weekAndDayLetter',
    // ...
};
```

### Features

Features are props with a `Feature` suffix: `stickyEventsFeature : false`, `eventTooltipFeature : { … }`, `printFeature : true`.

### Instance

Put a template `ref` on the component and read `ref.value.instance.value` for the Bryntum widget (e.g. `eventStore`, `features`). Inside `tbar` handlers, `source.up('<product>')` also works (e.g. `'gantt'`, `'scheduler'`).

### License key

8.x only. On 7.x, `references/v7.md` replaces this section.

Community needs none. Otherwise register it in `main.ts`, after the imports (they load the Bryntum packages, which create the `bryntum` global) and before mounting. `bryntum` is declared in no typings, so a `.ts`/`.tsx` file needs this declaration or `(window as any).bryntum.licenseKey` — see the `bryntum` skill, License key.

```typescript
import { createApp } from 'vue';
import App from './App.vue';

declare const bryntum : { licenseKey? : string };
bryntum.licenseKey = 'your-license-key';
createApp(App).mount('#app');
```

### Vite

8.x only. On 7.x, `references/v7.md` replaces this section.

Vite: see the `bryntum` skill; the names to add only on the loaded-multiple-times error are `'@bryntum/gantt-enterprise'` and `'@bryntum/gantt-vue-enterprise'`.

### Sizing

See the Sizing section of the `bryntum` skill — Vue Vite scaffolds use `#app` (not `#root`).
