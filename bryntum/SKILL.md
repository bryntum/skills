---
name: bryntum
description: >
  Build and integrate Bryntum components into web apps. Use this skill whenever the user is working with
  any Bryntum product — Scheduler, Scheduler Pro, Gantt, Calendar, Grid, or TaskBoard — regardless of
  whether they mention "Bryntum" by name. Trigger on phrases like "add a scheduler", "set up a gantt chart",
  "integrate a grid component", "calendar view", "task board", "resource scheduling", or any mention of
  @bryntum/* npm packages. Also trigger when the user asks about Bryntum CSS imports, themes, or
  framework wrappers (Angular, React, Vue). When in doubt, use this skill — it's better to consult it
  unnecessarily than to miss a Bryntum-specific rule and produce broken code.
metadata:
  tags: bryntum, scheduler, gantt, calendar, grid, taskboard, schedulerpro
---

## Step 1: Identify the product and version

Before writing any code, determine:

1. **Which product** is being integrated:

| Product | npm package | Trial package | CSS file |
|---|---|---|---|
| Gantt | `@bryntum/gantt` | `@bryntum/gantt-trial` | `gantt.css` |
| Scheduler | `@bryntum/scheduler` | `@bryntum/scheduler-trial` | `scheduler.css` |
| Scheduler Pro | `@bryntum/schedulerpro` | `@bryntum/schedulerpro-trial` | `schedulerpro.css` |
| Calendar | `@bryntum/calendar` | `@bryntum/calendar-trial` | `calendar.css` |
| Grid | `@bryntum/grid` | `@bryntum/grid-trial` | `grid.css` |
| TaskBoard | `@bryntum/taskboard` | `@bryntum/taskboard-trial` | `taskboard.css` |

2. **Which framework**: Vanilla JS, React, Angular, or Vue
3. **Version**: Check `package.json`. If unclear, use the docs tool to confirm the current major version.

If the `mcp__bryntum__search_bryntum_docs` tool is available, use it to look up product-specific configuration, API, and integration guides. Always pass the `product` and `version` parameters when you know them.

If the MCP tool is **not available**, look up documentation directly:
- API docs: `https://bryntum.com/products/{product}/docs/api/`
- Guides: `https://bryntum.com/products/{product}/docs/guide/`
- Example: `https://bryntum.com/products/calendar/docs/api/Calendar/view/Calendar`

Use `WebFetch` or `WebSearch` on `bryntum.com` to answer questions about config options, data models, and integration patterns.

---

## Installing packages

### Trial (no login required)

Trial packages are on the **public npm registry** — no `.npmrc`, no auth token, no Bryntum account needed.

Install using the npm alias pattern so imports use the standard `@bryntum/{product}` name:

```bash
npm install @bryntum/scheduler@npm:@bryntum/scheduler-trial@7.2.0
```

Or in `package.json`:

```json
"dependencies": {
  "@bryntum/scheduler": "npm:@bryntum/scheduler-trial@7.2.0"
}
```

The same pattern applies for all products — just swap `scheduler` for `gantt`, `calendar`, etc.

Framework wrappers (e.g. `@bryntum/scheduler-react`, `@bryntum/scheduler-angular`) have **no `-trial` suffix** and install directly from the public registry at the same version number.

> Use exact versions (no `^`). Trial adds a watermark but is otherwise fully functional.

### Licensed

Licensed packages live on the private Bryntum registry. Add to `.npmrc`:
```
@bryntum:registry=https://npm.bryntum.com
```
Then `npm login --registry=https://npm.bryntum.com` with Customer Zone credentials.

---

## Step 2: CSS setup (Bryntum 7+)

Bryntum 7 uses **plain CSS** — no SASS/SCSS. Follow this pattern for every product:

```css
/* FontAwesome icons (required — no longer auto-bundled) */
@import "@bryntum/{product}/fontawesome/css/fontawesome.css";
@import "@bryntum/{product}/fontawesome/css/solid.css";

/* Structural CSS (required) */
@import "@bryntum/{product}/{product}.css";

/* Theme CSS — svalbard-light is the default */
@import "@bryntum/{product}/svalbard-light.css";
```

**Available themes**: `svalbard-light`, `svalbard-dark`, `stockholm-light`, `stockholm-dark`, `material3-light`, `material3-dark`, `fluent2-light`, `fluent2-dark`

**If the project has a `node_modules` import path instead of a package alias**, use `./node_modules/@bryntum/...` — the pattern is identical, just a different prefix.

### Default font: Poppins

Bryntum hardcodes `Helvetica Neue, Arial, Helvetica, sans-serif` on `.b-widget` — there is **no `--b-font-family` CSS variable**. To apply a custom font, load it and override `.b-widget` directly:

```css
/* Load Poppins from Google Fonts */
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap");

/* Override Bryntum's hardcoded font */
.b-widget {
  font-family: 'Poppins', sans-serif;
}
```

**Default to Poppins** unless the user specifies otherwise. A `--b-font-family` variable does not exist and will have no effect.

### Full CSS example (Calendar, svalbard-light, Poppins)

```css
/* Google Fonts */
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap");

/* FontAwesome icons — required in Bryntum 7+ */
@import "@bryntum/calendar/fontawesome/css/fontawesome.css";
@import "@bryntum/calendar/fontawesome/css/solid.css";

/* Structural CSS — required */
@import "@bryntum/calendar/calendar.css";

/* Theme */
@import "@bryntum/calendar/svalbard-light.css";

.b-widget {
  font-family: 'Poppins', sans-serif;
}

html, body {
  height: 100%;
}

#app {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
```

### CSS rules — never break these

- Never use SASS/SCSS for Bryntum styling.
- Never use legacy single-file imports like `gantt.stockholm.css` — those are the old v5/v6 format.
- The theme file alone is **not enough**. Always pair it with the structural `{product}.css`.
- Default to `svalbard-light.css` unless the user asks for something else.
- Default to Poppins font via `.b-widget { font-family: ... }` — do not use `--b-font-family` (it doesn't exist).
- Use CSS variables for customization (`--b-widget-background`, etc.) rather than overriding specific class styles.
- Use normalized kebab-case class names: `.b-button-group`, not `.b-buttongroup`.

---

## Step 3: Framework-specific patterns

### Angular

- Install the Angular wrapper: `@bryntum/{product}-angular`
- Import `Bryntum{Product}Module` in your `AppModule` (or standalone component)
- Use the component tag: `<bryntum-{product}>`
- **When you add a new config property**, always bind it in `app.html` using Angular property binding:

```html
<bryntum-scheduler
  [resources]="schedulerProps.resources"
  [events]="schedulerProps.events"
  [columns]="schedulerProps.columns!"
></bryntum-scheduler>
```

- Config objects typically live in a `{product}Props` object in the component class.

### React

- Install the React wrapper: `@bryntum/{product}-react`
- Import `Bryntum{Product}` from `@bryntum/{product}-react`
- Pass config as props directly:

```tsx
<BryntumScheduler
  resources={resources}
  events={events}
  columns={columns}
/>
```

- Use `useRef` to get a reference to the underlying component instance when you need to call methods.

### Vue

- Install the Vue wrapper: `@bryntum/{product}-vue-3` (Vue 3) or `@bryntum/{product}-vue` (Vue 2)
- Register and use as a standard component
- Bind config via `v-bind` or individual props

### Vanilla JS

- Import the class directly: `import { {ProductClass} } from '@bryntum/{product}'`
- Instantiate with a config object and an `appendTo` / `adopt` target

---

## Step 4: Sizing

Bryntum components default to `100%` width and a minimum height of `10em`. To fill the screen, set parent containers explicitly:

```css
#app {
  margin: 0;
  display: flex;
  flex-direction: column;
  height: 100vh;
}
```

---

## Step 5: Look up what you don't know

Use `mcp__bryntum__search_bryntum_docs` if available, otherwise use `WebFetch` or `WebSearch` on `bryntum.com` for:
- Feature configuration (columns, resources, events, dependencies, etc.)
- API methods and events
- Migration guidance (e.g., CSS migration from v6 → v7)
- Framework-specific integration guides

When using the MCP tool, pass `product` (e.g., `"gantt"`, `"scheduler"`, `"schedulerpro"`, `"grid"`, `"calendar"`, `"taskboard"`) and `version` so results are scoped correctly.

---

## Quick checklist before finishing

- [ ] No SASS/SCSS
- [ ] No legacy single-file theme import (`{product}.stockholm.css`, etc.)
- [ ] FontAwesome imports present
- [ ] `{product}.css` comes before the theme CSS
- [ ] Theme is not used alone (always paired with structural CSS)
- [ ] Theme is `svalbard-light` if the user has not specified a preference
- [ ] Font is Poppins (via `.b-widget { font-family: 'Poppins', sans-serif }`) if the user has not specified a preference — never use `--b-font-family`
- [ ] Angular: new config props bound in template with `[prop]="..."`
- [ ] React: config passed as JSX props
- [ ] Component is sized correctly (parent has explicit height)
