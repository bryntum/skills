---
name: bryntum-react
description: >
  React-specific patterns for Bryntum components. Use this alongside the `bryntum` skill when
  building or integrating any Bryntum product (Scheduler, Gantt, Calendar, Grid, TaskBoard,
  Scheduler Pro) in a React project. Trigger when the project has React in package.json or when
  the user asks about React + Bryntum integration, JSX config, StrictMode issues, or
  @bryntum/*-react packages.
metadata:
  tags: bryntum, react, strictmode
---

## Quick-start guide

When scaffolding a new app, fetch (skip for migrations or existing apps):
`https://bryntum.com/products/{product}/docs-llm/guide/{Product}/quick-start/react.md`

---

## React

Framework wrapper package: `@bryntum/{product}-react`

### Component

Pass all config as JSX props. Use `useRef` for instance access:

```jsx
import { BryntumGantt } from '@bryntum/gantt-react';

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

### Gantt / Scheduler Pro: project data

Gantt and Scheduler Pro keep their data in a project. Don't put store data inside a `project` prop object — the wrapper logs *"Using the "project" prop with inner store configurations is not recommended"*. Put the data on the project component and pass its ref to the Gantt:

```tsx
import { useRef } from 'react';
import { BryntumGantt, BryntumGanttProjectModel } from '@bryntum/gantt-react';
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

Type `projectProps` as `BryntumGanttProjectModelProps`. Project-level settings (`calendar`, `calendars`, `loadUrl`, `autoLoad`) go in `projectProps` too. Scheduler Pro uses `BryntumSchedulerProProjectModel` the same way.

### Vite config

Include Bryntum packages in `optimizeDeps` to prevent multiple bundle loading in dev:

```js
optimizeDeps: {
    include: ['@bryntum/gantt', '@bryntum/gantt-react']
}
```

When adding to an **existing** `vite.config`, merge into the current `optimizeDeps.include` array — don't only check for this on fresh scaffolds, and don't overwrite an existing `optimizeDeps`.

### Sizing

See the Sizing section of the `bryntum` skill — for React, the app root selector is `#root`.

---

## Event / task editor

Default to Bryntum's built-in editor — **do not build a custom React/MUI dialog unless the user explicitly asks for one.** Removing fields (e.g. the Gantt `% Complete` / `Duration`), adding your own, and the supported way to swap in a custom dialog are all covered in the `bryntum-editor` skill.
