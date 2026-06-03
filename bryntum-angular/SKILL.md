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

Fetch before writing code:
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

Copy the demo's exported `…Props` config objects into a **separate** Bryntum config file (e.g. `gantt.config.ts`) imported by your component. **Do not overwrite the scaffold's `app.config.ts`** — that is Angular's `ApplicationConfig`, not a Bryntum file.

### Standalone component setup

Use standalone components (Angular 17+). Add the Bryntum module to `imports` in `@Component`:

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

```css
html, body { height: 100%; margin: 0; }
app-root { display: flex; flex: 1 1 100%; flex-direction: column; }
```
