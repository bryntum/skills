---
name: bryntum-react
description: >
  React-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Scheduler Pro, Gantt, Calendar, Grid,
  TaskBoard) in a React project, on the 8.0 tier packages (`@bryntum/<product>-react-<tier>`) or
  on 7.x. Trigger when the project has React in package.json or when the user asks about React +
  Bryntum integration, JSX config, StrictMode issues, or @bryntum/*-react packages.
metadata:
  tags: bryntum, react, strictmode, tier
---

## Version check — do this first

Detect the version as the `bryntum` skill's Version check says (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md). Every code block in this file is 8.0 code. 7.x → also load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-react/references/v7.md).

---

## Quick-start guide

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/<product>/docs-llm/guide/<Product>/quick-start/react.md`

On 8.x the page names the Enterprise packages; substitute the app's tier.

---

## React

### Packages and imports

8.x only. On 7.x, `references/v7.md` replaces this section.

Framework wrapper package: `@bryntum/<product>-react-<tier>`. It depends on `@bryntum/core-react-<tier>`, the wrappers of the products below it and the product package, so those arrive with it.

A wrapper exports its own product's components only. Import each thing from the package that owns it, and declare every package you import from in `package.json`:

```tsx
import { BryntumGantt, BryntumGanttProjectModel } from '@bryntum/gantt-react-enterprise';
import type { BryntumGanttProps, BryntumGanttProjectModelProps } from '@bryntum/gantt-react-enterprise';
import { BryntumButton, BryntumCombo } from '@bryntum/core-react-enterprise';   // Core widgets
import { BryntumGrid } from '@bryntum/grid-react-enterprise';                   // another product on the page
import { StringHelper } from '@bryntum/core-enterprise';                         // Core helpers
import type { TaskModel } from '@bryntum/gantt-enterprise';                      // product classes and types
```

Scheduler Pro is Scheduler at the Enterprise tier: `BryntumScheduler` and `BryntumSchedulerProjectModel` from `@bryntum/scheduler-react-enterprise`. `BryntumSchedulerPro` still exists but wraps the deprecated `SchedulerPro` class — don't use it in new code.

A Grid feature the product does not load itself (`cellTooltip`, `quickFind`) registers only after `import '@bryntum/grid-<tier>';` — see the `bryntum` skill, Imports.

### Component

8.x only. On 7.x, `references/v7.md` replaces this section.

Pass all config as JSX props. Use `useRef` for instance access:

```jsx
import { BryntumGantt } from '@bryntum/gantt-react-enterprise';

const ganttRef = useRef(null);
// access instance: ganttRef.current.instance
<BryntumGantt ref={ganttRef} {...ganttProps} />
```

### Features

Features are props with a `Feature` suffix, not a `features` object: `eventTooltipFeature={{ … }}`, `excelExporterFeature`. A `features` prop fails TypeScript with TS2353. At runtime they're still on `ref.current.instance.features`.

### StrictMode (React 18+)

React StrictMode double-mounts in dev (mount → unmount → mount). Bryntum configuration that needs to adapt depending on the component's state or props should be encapsulated in the component using the React `useState` hook to maintain reference across re-renders, prevent unnecessary calculations, and avoids side effects that need cleanup:

```javascript
const App = () => {
        const [ganttProps] = useState({
        startDate: start
        // Bryntum Gantt config options
    })
    return <BryntumGantt {...ganttProps} />;
};

createRoot(document.getElementById('root')!).render(
    <StrictMode><App /></StrictMode>
);
```

**Never retain references to destroyed Bryntum instances after unmount.** Do not use `useEffect`/`useRef` patterns that hold the instance across the unmount cycle.

### Next.js

Bryntum does not support server-side rendering: render the wrapper client-side only, from a component loaded with `dynamic(() => import('../components/Gantt'), { ssr : false })` (as the Next.js examples do).

### Gantt and Scheduler Enterprise: project data

8.x only. On 7.x, `references/v7.md` replaces this section.

Gantt and Scheduler Enterprise (the former Scheduler Pro) keep their data in a project. Don't put store data inside a `project` prop object — the wrapper logs *"Using the "project" prop with inner store configurations is not recommended"*. Put the data on the project component and pass its ref to the Gantt:

```tsx
import { useRef } from 'react';
import { BryntumGantt, BryntumGanttProjectModel } from '@bryntum/gantt-react-enterprise';
import { ganttProps, projectProps } from './ganttConfig'; // projectProps: { tasks, dependencies, ... }

function App() {
    const project = useRef<BryntumGanttProjectModel>(null);

    return (
        <>
            <BryntumGanttProjectModel ref={project} {...projectProps} />
            <BryntumGantt project={project} {...ganttProps} />
        </>
    );
}
```

Type `projectProps` as `BryntumGanttProjectModelProps`. Project-level settings (`calendar`, `calendars`, `loadUrl`, `autoLoad`) go in `projectProps` too. Scheduler Enterprise uses `BryntumSchedulerProjectModel` the same way.

### License key

8.x only. On 7.x, `references/v7.md` replaces this section.

Community needs none. Otherwise register it in `main.tsx`, after the imports (they load the Bryntum packages, which create the `bryntum` global) and before the first render. `bryntum` is declared in no typings, so a `.ts`/`.tsx` file needs this declaration or `(window as any).bryntum.licenseKey` — see the `bryntum` skill, License key.

```tsx
import { createRoot } from 'react-dom/client';
import App from './App';

declare const bryntum : { licenseKey? : string };
bryntum.licenseKey = 'your-license-key';
createRoot(document.getElementById('root')!).render(<App />);
```

**Next.js App Router:** there is no single client entry file. Set the key in the `"use client"` component that renders the Bryntum wrapper and is loaded with `ssr : false` (see Next.js above), after the wrapper import. A server module such as `app/layout.tsx` fails with `bryntum is not defined`.

### Vite

8.x only. On 7.x, `references/v7.md` replaces this section.

Vite: see the `bryntum` skill. On the loaded-multiple-times error, add every `@bryntum/*` package the app imports from, e.g. `'@bryntum/gantt-enterprise'`, `'@bryntum/gantt-react-enterprise'` and, if used, `'@bryntum/core-react-enterprise'`.

### Sizing

See the Sizing section of the `bryntum` skill — for React, the app root selector is `#root`.

---

## Event / task editor

Default to Bryntum's built-in editor — **do not build a custom React/MUI dialog unless the user explicitly asks for one.** Removing fields (e.g. the Gantt `% Complete` / `Duration`), adding your own, and the supported way to swap in a custom dialog are all covered in the `bryntum-editor` skill.
