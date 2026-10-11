---
name: bryntum-angular
description: >
  Angular-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Gantt, Calendar, Grid, TaskBoard,
  Scheduler Pro) in an Angular project. Trigger when the project has Angular in package.json or
  when the user asks about Angular + Bryntum integration, template binding, standalone components,
  or @bryntum/*-angular packages.
metadata:
  tags: bryntum, angular, standalone
---

## Quick-start guide

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/{product}/docs-llm/guide/{Product}/quick-start/angular.md`

---

## Angular

Framework wrapper package: `@bryntum/{product}-angular`

### Component selector

Use the kebab-case selector, e.g. `<bryntum-gantt>`, `<bryntum-scheduler>`, `<bryntum-grid>`.

### Binding props

Every config prop **must** be bound with `[prop]="..."` in the template — not as a plain attribute:

```html
<bryntum-gantt
    [columns]="ganttConfig.columns"
    [project]="ganttConfig.project"
    [viewPreset]="ganttConfig.viewPreset">
</bryntum-gantt>
```

### Config separation

Copy the demo's exported `…Props` config objects into a **separate** Bryntum config file (e.g. `gantt.config.ts`) imported by your component. **Do not overwrite the scaffold's `app.config.ts`** in a freshly scaffolded standalone app — that is Angular's `ApplicationConfig`, not a Bryntum file. In existing apps, check the file's contents first: older NgModule apps often keep the Bryntum config there.

### Standalone component setup

For new apps, use standalone components (Angular 17+). Add the Bryntum module to `imports` in `@Component`. In an existing NgModule app, keep the Bryntum module in the NgModule's `imports`; a Bryntum upgrade isn't the time to convert to standalone.

```typescript
import { BryntumGanttModule } from '@bryntum/gantt-angular';

@Component({
    standalone : true,
    imports    : [BryntumGanttModule],
    templateUrl: './app.component.html',
})
export class AppComponent {
    ganttConfig = { ... };
}
```

### Sizing

See the Sizing section of the `bryntum` skill for the general rule. Angular's `app-root` is a block element, so use a flex layout rather than `height: 100%`:

```css
html, body { height: 100%; margin: 0; }
app-root { display: flex; flex: 1 1 100%; flex-direction: column; }
```

### Production build warning

`ng build` warns `2 rules skipped due to selector errors` (`:has()`, `:host(:not(.b-nothing))`) with Bryntum 7 CSS. The theme variables are still emitted, so the warning is expected.
