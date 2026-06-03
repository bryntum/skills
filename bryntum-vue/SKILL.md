---
name: bryntum-vue
description: >
  Vue 3-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Gantt, Calendar, Grid, TaskBoard,
  Scheduler Pro) in a Vue 3 project. Trigger when the project has Vue in package.json or when
  the user asks about Vue + Bryntum integration, v-bind, BryntumXxxProps, or
  @bryntum/*-vue-3 packages.
metadata:
  tags: bryntum, vue, vue3
---

## Quick-start guide

Fetch before writing code:
`https://bryntum.com/products/{product}/docs-llm/guide/{Product}/quick-start/vue-3.md`

---

## Vue 3

Framework wrapper package: `@bryntum/{product}-vue-3`

### Component

Spread config with `v-bind`:

```vue
<bryntum-gantt v-bind="ganttConfig" />
```

Or bind individual props:

```vue
<bryntum-gantt :columns="columns" :project="project" />
```

Register the component globally or per-component:

```typescript
// main.ts — global
import { BryntumVue3Plugin } from '@bryntum/gantt-vue-3';
app.use(BryntumVue3Plugin);

// or per-component
import { BryntumGantt } from '@bryntum/gantt-vue-3';
```

### Type-safe v-bind

Type the config object as `BryntumGanttProps` so `v-bind` type-checks:

```typescript
import type { BryntumGanttProps } from '@bryntum/gantt-vue-3';

const ganttConfig: BryntumGanttProps = {
    viewPreset : 'weekAndDayLetter',
    // ...
};
```

### Sizing

Vue Vite scaffolds use `#app` (not `#root`):

```css
html, body, #app { height: 100%; margin: 0; }
```
