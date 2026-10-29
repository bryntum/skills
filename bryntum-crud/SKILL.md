---
name: bryntum-crud
description: >
  Backend, CrudManager, and AjaxStore patterns for Bryntum components. Use this alongside the
  `bryntum` skill when the user needs to connect a Bryntum product to a backend — loading data
  from an API, persisting changes, or setting up sync. Trigger on phrases like "connect to a
  backend", "save data", "load from server", "CrudManager", "project transport", "AjaxStore",
  "sync", "REST API", or "database" in a Bryntum project context.
metadata:
  tags: bryntum, crud, crudmanager, project, transport, ajaxstore, backend, sync
---

## Version check — do this first

Read the app's `package.json` and pick the branch (in a monorepo, the workspace package that will import Bryntum). Every code block in this file is 8.0 code.

| `@bryntum/*` dependencies | Version | Do |
|---|---|---|
| Any tiered name: `@bryntum/<product>-community`, `-business` or `-enterprise` (untiered names next to it, other than `@bryntum/demo-resources`, are migration leftovers: finish with `bryntum-migrate`) | **8.0** | Follow this file |
| Only untiered names: `@bryntum/<product>`, a wrapper without a tier suffix, `-trial`, `-thin`, `-lib` or `@bryntum/schedulerpro` (ignore `@bryntum/demo-resources`) | **7.x or older** | Load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-crud/references/v7.md). Read the installed version from `node_modules/@bryntum/<product>/package.json` (`<product>-thin` for a thin app), or from the lockfile when nothing is installed; below 7.0, say this skill covers 7.x and 8.x and offer `bryntum-migrate`. Don't upgrade unless asked — that is `bryntum-migrate` |
| None, and no `build/` folder | **New app on 8.0** | Follow this file |
| No `@bryntum/*` dependency but a `build/` folder (distribution archive, see Workspace classification in the `bryntum` skill) | **from `build/package.json`** | `version` 8.x: follow this file. 7.x: load `references/v7.md` |

---

## Backend / CrudManager / AjaxStore

### Grid: AjaxStore (no CrudManager)

Grid uses `AjaxStore` directly — it has no CrudManager. Configure on the store:

```javascript
store: {
    readUrl  : '/api/records',
    createUrl: '/api/records',
    updateUrl: '/api/records',
    deleteUrl: '/api/records',
}
```

See the AjaxStore guide: `https://bryntum.com/products/grid/docs/guide/Grid/data/ajaxstore`

### All other products: the project is the CrudManager

8.x only. On 7.x, `references/v7.md` replaces this section.

Scheduler, Gantt, Calendar and TaskBoard load and sync through the **project**, which has CrudManager built in on every tier. Configure the transport on the project (`loadUrl` / `syncUrl` are shortcuts for `transport.load.url` / `transport.sync.url`):

```javascript
project : {
    loadUrl  : '/api/load',
    syncUrl  : '/api/sync',
    autoLoad : true,
    autoSync : true      // optional: sync after every change
}
```

HTTP methods default to GET (load) and POST (sync) and are configurable via the `transport` config. In React pass the same object as the `project` prop; a transport-only `project` prop is fine (see Data loading in the `bryntum` skill).

The `crudManager` config and the standalone `CrudManager` class are **deprecated** (removed in 9.0) and log a console warning. `scheduler.crudManager` now returns the project, which carries the whole CrudManager API (`load()`, `sync()`, `changes`, `acceptChanges()`, `revertChanges()`, `hasChanges()`). Never configure transport on both the project and a `crudManager` — that throws.

See the CrudManager guide: `https://bryntum.com/products/gantt/docs/guide/Gantt/data/crud_manager_project`

**Never combine a `project` prop with inline data props** — it throws. Pick one approach.

### No backend? Use demo-style static data

When adding a Bryntum component to an app that has **no API**, do NOT reach for CrudManager/AjaxStore — and do not try to fake persistence with localStorage. Follow the Bryntum **demos**: provide example data either inline or from a static `data.json` file loaded at startup.

```jsx
// inline (Scheduler Enterprise / Gantt: put the data on the project component instead — see Data loading in the `bryntum` skill)
<BryntumScheduler events={seedEvents} resources={seedResources} />

// or load a static JSON file (shape matches the demos: { events: [...], resources: [...] })
const data = await fetch('/data.json').then(r => r.json());
```

This gets a working, populated component on screen. If the user actually needs edits to persist across reloads, that requires a backend — wire `loadUrl`/`syncUrl` as shown above; don't substitute client-side storage.

### Phantom IDs

When the backend creates a new record, it must return the mapping from the client-generated `$PhantomId` to the real server ID, for all related records in the same sync response. Missing this causes foreign key mismatches on the next sync.

### Partial sync

Sync requests send only changed fields, not full records. Build SQL `SET` clauses dynamically from the request body — don't assume all columns are present.

### Common data gotchas

- `exceptionDates` must be `[]`, never `null`
- `allDay` must be a boolean (`true`/`false`), not `0`/`1`
- Use `assignments` store rather than `resourceId` on events when an event can belong to multiple resources

### Dev setup with proxy

When using `server.proxy` in Vite to forward API calls to a local backend, start the backend **before** running `npm run dev` — Vite's proxy resolves at startup.
