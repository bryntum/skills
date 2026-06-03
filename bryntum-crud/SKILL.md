---
name: bryntum-crud
description: >
  Backend, CrudManager, and AjaxStore patterns for Bryntum components. Use this alongside the
  `bryntum` skill when the user needs to connect a Bryntum product to a backend — loading data
  from an API, persisting changes, or setting up sync. Trigger on phrases like "connect to a
  backend", "save data", "load from server", "CrudManager", "AjaxStore", "sync", "REST API",
  or "database" in a Bryntum project context.
metadata:
  tags: bryntum, crud, crudmanager, ajaxstore, backend, sync
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

### All other products: CrudManager

Scheduler, Scheduler Pro, Gantt, Calendar, and TaskBoard use CrudManager. Wire it via the `crudManager` prop (or inside a `project`):

```javascript
crudManager: {
    loadUrl : '/api/load',
    syncUrl : '/api/sync',
}
```

`loadUrl` and `syncUrl` are shortcuts for `transport.load.url` / `transport.sync.url`. HTTP methods default to GET (load) and POST (sync) but are configurable via the `transport` config.

See the CrudManager guide: `https://bryntum.com/products/gantt/docs/guide/Gantt/data/crud_manager`

**Never combine a `crudManager`/`project` prop with inline data props** — it throws. Pick one approach.

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
