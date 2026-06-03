---
name: bryntum-vanilla
description: >
  Vanilla JS patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Gantt, Calendar, Grid, TaskBoard,
  Scheduler Pro) without a framework — plain JavaScript or TypeScript with no React/Angular/Vue.
  Trigger when there is no framework in package.json or when the user asks about vanilla JS
  Bryntum integration or direct class instantiation.
metadata:
  tags: bryntum, vanilla, javascript
---

## Vanilla JS

No framework wrapper package needed — import directly from the core package.

### Import and instantiate

```javascript
import { Gantt } from '@bryntum/gantt';

const gantt = new Gantt({
    appendTo : 'app',
    columns  : [...],
    // ...
});
```

Always import from `@bryntum/{product}` (the installed package or alias). **Never** use the demo's relative build paths like `../../build/gantt.module.js`.

### CSS

Import in the app's entry stylesheet — same order as all other frameworks:

```css
@import "@bryntum/gantt/fontawesome/css/fontawesome.css";
@import "@bryntum/gantt/fontawesome/css/solid.css";
@import "@bryntum/gantt/gantt.css";
@import "@bryntum/gantt/svalbard-light.css";
```

### Sizing

```css
html, body, #app { height: 100%; margin: 0; }
```

The `appendTo` target element also needs an explicit height if it does not inherit from `#app`.
