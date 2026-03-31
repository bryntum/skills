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

## Products

| Product | npm package | Trial package | CSS file |
|---|---|---|---|
| Gantt | `@bryntum/gantt` | `@bryntum/gantt-trial` | `gantt.css` |
| Scheduler | `@bryntum/scheduler` | `@bryntum/scheduler-trial` | `scheduler.css` |
| Scheduler Pro | `@bryntum/schedulerpro` | `@bryntum/schedulerpro-trial` | `schedulerpro.css` |
| Calendar | `@bryntum/calendar` | `@bryntum/calendar-trial` | `calendar.css` |
| Grid | `@bryntum/grid` | `@bryntum/grid-trial` | `grid.css` |
| TaskBoard | `@bryntum/taskboard` | `@bryntum/taskboard-trial` | `taskboard.css` |

Before writing code: identify **product**, **framework** (React/Angular/Vue/Vanilla), and **version** (check `package.json`).

Use `mcp__bryntum__search_bryntum_docs` (pass `product` + `version`) for API/config lookups. If unavailable, ask the user to add it:
```bash
claude mcp add --transport http bryntum https://mcp.bryntum.com
```

**Claude Desktop** (add to `claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "bryntum": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.bryntum.com"]
    }
  }
}
```

Fallback: `WebFetch`/`WebSearch` on `bryntum.com/products/{product}/docs/`.

---

## Installing packages

**Trial** (public npm, no auth):
```bash
npm install @bryntum/gantt@npm:@bryntum/gantt-trial@7.2.0
```
Framework wrappers have no `-trial` suffix: `npm install @bryntum/gantt-react@7.2.0`. Use exact versions (no `^`).

**Licensed**: Add `@bryntum:registry=https://npm.bryntum.com` to `.npmrc`, then `npm login --registry=https://npm.bryntum.com`.

---

## CSS setup (v7+)

Bryntum 7 uses **plain CSS only** — no SASS/SCSS. Three imports required in order:

```css
@import "@bryntum/{product}/fontawesome/css/fontawesome.css";  /* icons */
@import "@bryntum/{product}/fontawesome/css/solid.css";
@import "@bryntum/{product}/{product}.css";                    /* structural — required */
@import "@bryntum/{product}/svalbard-light.css";               /* theme */
```

**Themes**: `svalbard-light` (default), `svalbard-dark`, `stockholm-light`, `stockholm-dark`, `visby-light`, `visby-dark`, `material3-light`, `material3-dark`, `fluent2-light`, `fluent2-dark`

**Font**: Default to Poppins. No `--b-font-family` variable exists — override `.b-widget` directly:
```css
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600&display=swap");
.b-widget { font-family: 'Poppins', sans-serif; }
```

**CSS rules**:
- Never SASS/SCSS. Never legacy single-file imports (`gantt.stockholm.css`).
- Structural `{product}.css` must always accompany the theme file.
- Use CSS variables (`--b-widget-background`, etc.) for customization.

---

## Dynamic theme switching (dark mode)

**Must** use `DomHelper.setTheme()` with `<link>` tags. CSS `@import` will NOT work.

**1. Load CSS via `<link>` in `index.html`** (not CSS `@import`):
```html
<link rel="stylesheet" href="/node_modules/@bryntum/{product}/fontawesome/css/fontawesome.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/{product}/fontawesome/css/solid.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/{product}/{product}.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/{product}/svalbard-light.css" data-bryntum-theme />
```
The `data-bryntum-theme` attribute is **required** — `setTheme()` finds and swaps this `<link>`.

**2. Call `DomHelper.setTheme()`**:
```javascript
import { DomHelper } from '@bryntum/{product}';
DomHelper.setTheme('svalbard-dark');  // or 'svalbard-light'
```

**What does NOT work**:
- CSS `@import` + `setTheme()` → no `<link>` to target, silently fails
- Manual CSS overrides (`.dark .b-gridbase { ... }`) → only covers a fraction of theme variables

**React tbar example**:
```jsx
const ganttConfig = useMemo(() => ({
  tbar: {
    items: [
      { type: 'widget', html: '<h2 style="margin:0">Title</h2>' },
      '->',
      {
        type: 'button', icon: 'b-fa b-fa-moon', text: 'Dark Mode',
        toggleable: true,
        onToggle: ({ pressed }) => {
          DomHelper.setTheme(pressed ? 'svalbard-dark' : 'svalbard-light');
        },
      },
    ],
  },
}), []);
```

---

## Framework wrappers

**React**: `@bryntum/{product}-react` → `<BryntumGantt {...config} />`. Use `useRef` for instance access.

**Angular**: `@bryntum/{product}-angular` → `<bryntum-gantt [columns]="props.columns" />`. Bind new props in template with `[prop]="..."`.

**Vue**: `@bryntum/{product}-vue-3` (Vue 3) or `@bryntum/{product}-vue` (Vue 2). Bind via `v-bind` or individual props.

**Vanilla**: `import { Gantt } from '@bryntum/gantt'` → instantiate with config + `appendTo`.

---

## Data loading

**Never mix `project` prop with inline data props** — throws an error. Pick one:

```tsx
// ✅ Data inside project config (recommended)
<BryntumGantt project={{ tasks: myTasks, dependencies: myDeps }} />

// ✅ Data as props, no project
<BryntumGantt tasks={myTasks} dependencies={myDeps} />

// ❌ WRONG — will throw
<BryntumGantt tasks={myTasks} project={{ autoSetConstraints: true }} />
```

**v7 deprecations**: Use `tasks`/`dependencies`/`resources`/`assignments` (not `tasksData`/`dependenciesData` etc. — deprecated in v7).

---

## Sizing

Components default to `100%` width, `10em` min-height. Set parent height explicitly:
```css
#app { height: 100vh; display: flex; flex-direction: column; }
```

---

## Backend / CrudManager

- `loadUrl` (GET by default) and `syncUrl` (POST by default) are convenience shortcuts for `transport.load.url` and `transport.sync.url`; HTTP methods are configurable via the `transport` config
- Prefer `assignments` store over `resourceId` on events
- Phantom IDs (`$PhantomId`): backend must resolve phantom→real ID mapping for related records in the same sync
- `exceptionDates` must be `[]`, never `null`; `allDay` must be boolean (not `0`/`1`)
- Sync updates are **partial** — only changed fields sent. Build SET clauses dynamically
- Start backend before Vite when using `server.proxy`

---

## React StrictMode

React 18+ StrictMode double-mounts components in dev (mount → unmount → mount). This can cause data loading issues with Bryntum components. If this happens, wait for data to load before rendering the component. Do not retain references to destroyed Bryntum instances after unmount. Bryntum's official basic React example sidesteps the issue by using `useState` for config instead of `useEffect`/ref — `useState` preserves the config across the remount cycle, so there are no side effects to clean up and no refs that could point to a destroyed instance:

```javascript
const App = () => {
    const [ganttProps] = useState(useGanttProps());
    return <BryntumGantt {...ganttProps} />;
};

createRoot(document.getElementById('root')!).render(
    <StrictMode><App /></StrictMode>
);
```

---

## Scaffolding safety

**NEVER `rm -rf` the project directory** to re-scaffold — destroys config files. Scaffold in-place or add files manually.

---

## Checklist

- [ ] No SASS/SCSS, no legacy single-file theme imports
- [ ] FontAwesome + structural CSS + theme CSS (in order)
- [ ] Default theme: `svalbard-light`; default font: Poppins via `.b-widget`
- [ ] No mixing `project` prop with inline data props
- [ ] Using `tasks`/`dependencies`/`resources` (not deprecated `*Data` names)
- [ ] Parent has explicit height for proper sizing
- [ ] Dark mode: CSS via `<link>` + `data-bryntum-theme`, swap via `DomHelper.setTheme()`
- [ ] React: don't retain references to destroyed Bryntum instances after StrictMode unmount
- [ ] Angular: new props bound in template with `[prop]="..."`
