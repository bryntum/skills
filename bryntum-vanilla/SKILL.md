---
name: bryntum-vanilla
description: >
  Vanilla JS patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Scheduler Pro, Gantt, Calendar, Grid,
  TaskBoard) without a framework — plain JavaScript or TypeScript with no React/Angular/Vue, on
  the 8.0 tier packages (`@bryntum/<product>-<tier>`) or on 7.x. Trigger when there is no
  framework in package.json or when the user asks about vanilla JS Bryntum integration or direct
  class instantiation.
metadata:
  tags: bryntum, vanilla, javascript, tier
---

## Version check — do this first

Detect the version as the `bryntum` skill's Version check says (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md). Every code block in this file is 8.0 code. 7.x → also load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-vanilla/references/v7.md).

---

## Quick-start guide

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/<product>/docs-llm/guide/<Product>/quick-start/javascript-npm.md`

On 8.x the page names the Enterprise packages; substitute the app's tier.

---

## Vanilla JS

### Packages and imports

8.x only. On 7.x, `references/v7.md` replaces this section.

No wrapper package. The product package `@bryntum/<product>-<tier>` depends on Core and the products it is built on, so those arrive with it.

A package exports its own classes only. Import each class from the package that owns it (the product class itself: see Import and instantiate), and declare every package you import from in `package.json`:

```javascript
import { StringHelper, Toast, Button } from '@bryntum/core-enterprise';   // Core helpers and widgets
import { Column } from '@bryntum/grid-enterprise';                          // another product's classes
```

### Import and instantiate

8.x only. On 7.x, `references/v7.md` replaces this section.

```javascript
import { Gantt } from '@bryntum/gantt-enterprise';

const gantt = new Gantt({
    appendTo : 'app',
    columns  : [...],
    // ...
});
```

Always import from the installed `@bryntum/<product>-<tier>` package. **Never** use the demo's relative build paths like `../../build/gantt.module.js`.

### License key

8.x only. On 7.x, `references/v7.md` replaces this section.

Community needs none. Otherwise register it in the entry file (`main.js`), after the imports (they create the `bryntum` global) and before the first component. `bryntum` is declared in no typings, so a `.ts` file needs `declare const bryntum : { licenseKey? : string };` or `(window as any).bryntum.licenseKey` — see the `bryntum` skill, License key.

```javascript
// main.js, after the import shown in Import and instantiate
bryntum.licenseKey = 'your-license-key';

const gantt = new Gantt({ appendTo : 'app', /* ... */ });
```

Or, before the module script in `index.html`: `<script>window.bryntum = { licenseKey : 'your-license-key' };</script>`.

### CSS

8.x only. On 7.x, `references/v7.md` replaces this section.

Import in the app's entry stylesheet — same order as all other frameworks. Font Awesome comes from Core, the structural sheet and the theme from the product package:

```css
@import "@bryntum/core-enterprise/fontawesome/css/fontawesome.css";
@import "@bryntum/core-enterprise/fontawesome/css/solid.css";
@import "@bryntum/gantt-enterprise/gantt.css";
@import "@bryntum/gantt-enterprise/svalbard.css";
```

Several products on one page: the thin sheets instead — see the `bryntum` skill, CSS setup.

### Sizing

See the Sizing section of the `bryntum` skill — vanilla scaffolds use `#app`. Note: the `appendTo` target element also needs an explicit height if it does not inherit from `#app`.
