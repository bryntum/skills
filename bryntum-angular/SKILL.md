---
name: bryntum-angular
description: >
  Angular-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Scheduler Pro, Gantt, Calendar, Grid,
  TaskBoard) in an Angular project, on the 8.0 tier packages (`@bryntum/<product>-angular-<tier>`)
  or on 7.x. Trigger when the project has Angular in package.json or when the user asks about
  Angular + Bryntum integration, template binding, standalone components, or @bryntum/*-angular
  packages.
metadata:
  tags: bryntum, angular, standalone, tier
---

## Version check — do this first

Read the app's `package.json` and pick the branch (in a monorepo, the workspace package that will import Bryntum). Every code block in this file is 8.0 code.

| `@bryntum/*` dependencies | Version | Do |
|---|---|---|
| Any tiered name: `@bryntum/<product>-community`, `-business` or `-enterprise` (untiered names next to it, other than `@bryntum/demo-resources`, are migration leftovers: finish with `bryntum-migrate`) | **8.0** | Follow this file |
| Only untiered names: `@bryntum/<product>`, a wrapper without a tier suffix, `-trial`, `-thin`, `-lib` or `@bryntum/schedulerpro` (ignore `@bryntum/demo-resources`) | **7.x or older** | Load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-angular/references/v7.md). Read the installed version from `node_modules/@bryntum/<product>/package.json` (`<product>-thin` for a thin app), or from the lockfile when nothing is installed; below 7.0, say this skill covers 7.x and 8.x and offer `bryntum-migrate`. Don't upgrade unless asked — that is `bryntum-migrate` |
| None, and no `build/` folder | **New app on 8.0** | Follow this file |
| No `@bryntum/*` dependency but a `build/` folder (distribution archive, see Workspace classification in the `bryntum` skill) | **from `build/package.json`** | `version` 8.x: follow this file. 7.x: load `references/v7.md` |

---

## Quick-start guide

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/<product>/docs-llm/guide/<Product>/quick-start/angular.md`

On 8.x the page names the Enterprise packages; substitute the app's tier.

---

## Angular

### Packages and imports

8.x only. On 7.x, `references/v7.md` replaces this section.

Framework wrapper package: `@bryntum/<product>-angular-<tier>`. **Angular 15 or newer is required**; the View Engine package `-angular-view` is no longer published. The wrapper depends on `@bryntum/core-angular-<tier>`, the wrappers of the products below it and the product package, so those arrive with it.

Each wrapper ships the module and components of its own product only. Import from the package that owns what you use, and declare every package you import from in `package.json`:

| You use | Import |
|---|---|
| The product | `BryntumGanttModule`, `BryntumGanttComponent`, `BryntumGanttProjectModelComponent` from `@bryntum/gantt-angular-<tier>` |
| Core widgets in templates (`<bryntum-button>`, `<bryntum-combo>`, ...) | `BryntumCoreModule` from `@bryntum/core-angular-<tier>`, next to the product module |
| Another product on the page | that product's module, e.g. `BryntumGridModule` from `@bryntum/grid-angular-<tier>` |
| Config types, helpers, model classes | `BryntumGanttProps` from the wrapper; `StringHelper` etc. from `@bryntum/core-<tier>`; `TaskModel` from `@bryntum/gantt-<tier>` |

Scheduler Pro is Scheduler at the Enterprise tier: `BryntumSchedulerModule` and `BryntumSchedulerComponent` from `@bryntum/scheduler-angular-enterprise`. `BryntumSchedulerProComponent` still exists but wraps the deprecated `SchedulerPro` class, which logs a deprecation — use `BryntumSchedulerComponent`.

A Grid feature the product does not load itself (`cellTooltip` or `quickFind` in a Scheduler or Gantt) registers only when the Grid package is loaded: import `BryntumGridModule` next to the product's own module, or `import '@bryntum/grid-<tier>';` — see the `bryntum` skill, Imports.

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

### SSR

Bryntum renders in the browser only: disable SSR when scaffolding, or in an SSR app render the Bryntum component client-side only.

### Standalone component setup

8.x only. On 7.x, `references/v7.md` replaces this section.

For new apps, use standalone components (Angular 17+). Add the Bryntum module to `imports` in `@Component`. In an existing NgModule app, keep the Bryntum module in the NgModule's `imports`; a Bryntum upgrade isn't the time to convert to standalone.

```typescript
import { BryntumGanttModule } from '@bryntum/gantt-angular-enterprise';

@Component({
    standalone : true,
    imports    : [BryntumGanttModule],
    templateUrl: './app.component.html',
})
export class AppComponent {
    ganttConfig = { ... };
}
```

### License key

8.x only. On 7.x, `references/v7.md` replaces this section.

Community needs none. Otherwise register it in `main.ts`, after the imports (they load the Bryntum packages, which create the `bryntum` global) and before bootstrapping. `bryntum` is declared in no typings, so a `.ts`/`.tsx` file needs this declaration or `(window as any).bryntum.licenseKey` — see the `bryntum` skill, License key.

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

declare const bryntum : { licenseKey? : string };
bryntum.licenseKey = 'your-license-key';
bootstrapApplication(AppComponent, appConfig);
```

### Sizing

See the Sizing section of the `bryntum` skill for the general rule. Angular's `app-root` is a block element, so use a flex layout rather than `height: 100%`:

```css
html, body { height: 100%; margin: 0; }
app-root { display: flex; flex: 1 1 100%; flex-direction: column; }
```

### Production build warning

`ng build` may warn `rules skipped due to selector errors` (`:has()`, `:host(:not(.b-nothing))`) with Bryntum CSS. The theme variables are still emitted, so the warning is expected.
