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

Fetch before writing code:
`https://bryntum.com/products/{product}/docs-llm/guide/{Product}/quick-start/react.md`

---

## React

Framework wrapper package: `@bryntum/{product}-react`

### Component

```jsx
import { BryntumGantt } from '@bryntum/gantt-react';

const App = () => {
    const [ganttProps] = useState(useGanttProps());
    return <BryntumGantt {...ganttProps} />;
};
```

Pass all config as JSX props. Use `useRef` for instance access:

```jsx
const ganttRef = useRef(null);
// access instance: ganttRef.current.instance
<BryntumGantt ref={ganttRef} {...ganttProps} />
```

### StrictMode (React 18+)

React StrictMode double-mounts in dev (mount → unmount → mount). Use `useState` for config — it preserves the config object across the remount cycle, avoiding side effects that need cleanup:

```javascript
const App = () => {
    const [ganttProps] = useState(useGanttProps());
    return <BryntumGantt {...ganttProps} />;
};

createRoot(document.getElementById('root')!).render(
    <StrictMode><App /></StrictMode>
);
```

**Never retain references to destroyed Bryntum instances after unmount.** Do not use `useEffect`/`useRef` patterns that hold the instance across the unmount cycle.

### Sizing

`html`, `body`, and `#root` all need an explicit height:

```css
html, body, #root { height: 100%; margin: 0; }
```

### Vite config

Include Bryntum packages in `optimizeDeps` to prevent multiple bundle loading in dev:

```js
optimizeDeps: {
    include: ['@bryntum/gantt', '@bryntum/gantt-react']
}
```
