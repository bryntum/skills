# Data migration: models, stores, loading, saving

Tags as in `api-mapping.md`. For CrudManager/AjaxStore mechanics (phantom IDs, partial sync, common data gotchas), also
load the `bryntum-crud` skill.

## Pick one strategy per app and record it in MIGRATION_PLAN.md

1. **Map fields in the model** — the server and its JSON stay unchanged; Bryntum fields read from the old names via
   `dataSource`. Default when a real backend exists that the team doesn't want to change. (G-S, G-G)
2. **Convert the data** to Bryntum's camelCase fields and structures. Required for structures that can't be expressed
   as field mappings (Gantt calendars, `TaskType`, baselines). For static JSON, convert with a checked-in script
   (e.g. `scripts/convert-data.mjs`) so it can be re-run. With a real backend, this means changing the server, so say
   so in the plan and get the user's agreement. (G-G)

Count the important records (resources, events/tasks, assignments, dependencies, rows) before and after, and record
both counts in the report.

## Ext models → Bryntum models

| Ext | Bryntum | Tag |
|---|---|---|
| `Ext.define('M', { extend : 'Ext.data.Model', fields : [...] })` | `class M extends Model { static $name = 'M'; static fields = [...] }` | SRC |
| field `mapping : 'Name'` | `dataSource : 'Name'` (dot paths allowed for nested data) | G-S |
| field `type : 'date'`, `dateFormat` | `type : 'date'`, `format` (moment-style tokens) | DOC |
| field `type : 'int'` / `'float'` / `'number'` | `type : 'int'` / `'number'` | DOC |
| field `type : 'bool'` / `'boolean'` | `type : 'boolean'` | DOC |
| field `defaultValue` | `defaultValue` | SRC |
| field `calculate` / `convert` | `calculate : r => ...` for derived fields (SRC); `convert` — verify (UNV). Not on `startDate`/`endDate` (epoch ms: `api-mapping.md` §3) | SRC / UNV |
| TypeScript: `useDefineForClassFields : false` (older Vite templates) | leave it at the ES2022 default (`true`): `static fields` / `static configurable` throw "has only a getter" otherwise | TEST |
| `idProperty : 'Id'` | `static idField = 'Id'` on the model subclass, or `{ name : 'id', dataSource : 'Id' }`. `idField` is **not** a Store config (silently ignored there; the official Scheduler guide is wrong) | SRC |
| `hasMany` / `belongsTo` associations | no direct equivalent — flatten, or use Bryntum's built-in relations (assignments, dependencies, tree children) | UNV |
| Sch/Gnt model subclasses (`Sch.model.Event`, `Gnt.model.Task`) | `EventModel`, `ResourceModel`, `TaskModel`, `DependencyModel`, `AssignmentModel`, `CalendarModel` | G-S, G-G |

Leave the `id` field untyped, even when Ext declared `{ name : 'Id', type : 'int' }`; JSON numeric ids stay numbers
anyway (8.0.0, until bryntum/bryntum#14232 is fixed: a typed `id` makes a grouped store return wrong record counts).

After a `dataSource` mapping the record has only the Bryntum field: `fwa.get('scheduledCrewId')` is `undefined`, read
`resourceId` (a ported "crew mismatch" check silently never fired). The source key is not a property of the record.

Extra fields that keep their name can be declared as strings: `fields : ['location', { name : 'eventType', defaultValue : 'appointment' }]`
(SRC). Declare only fields the app actually uses.

## Ext stores and proxies → Bryntum loading

| Ext | Bryntum | Tag |
|---|---|---|
| store with inline `data` | `data` / `events` / `resources` / `tasks` config (never the deprecated `*Data` props) | SRC |
| `proxy : { type : 'ajax', url }` + `autoLoad` (Grid) | `store : { readUrl, autoLoad : true }` (AjaxStore). Writes: `createUrl`, `updateUrl`, `deleteUrl` | SRC |
| `proxy : { type : 'rest', url }` | AjaxStore with per-operation URLs + `httpMethods` — verify request shape against the server | DOC |
| `reader : { rootProperty : 'data' }` | AjaxStore `responseDataProperty` | DOC |
| `remoteSort` / `remoteFilter` / paging (`pageSize`) | AjaxStore `remoteSort` / `remoteFilter` / `remotePaging` + `pageSize` | DOC |
| `sorters`, `groupers`, `filters` | same names on the store. For computed groups use a calculated field (a function `field` in `groupers` throws on 8.0.0, until bryntum/bryntum#14230 is fixed). Ext **re-sorts on add**, Bryntum appends: set `reapplySortersOnAdd : true` on the store. Ext `remoteSort : true` against a server that ignores `sort`/`dir` left the list unsorted; Bryntum sorts locally (a behavior fix to report) | SRC, TEST |
| `store.sync()` | AjaxStore `commit()`; CrudManager `sync()` | DOC |
| Sch/Gnt `CrudManager` (`transport`, `load`, `sync`) | Bryntum `project` (every product and tier in 8.0: the project has the CrudManager built in; the standalone `CrudManager` class and the `crudManager` config are deprecated and go in 9.0) with `loadUrl`/`syncUrl` or `transport` | G-S, G-G |
| separate event/resource stores loaded by separate proxies (Scheduler) | one `project` load for both (`loadUrl` with `{ resources : { rows }, events : { rows } }`), or one store each: `project : { resourceStore : { readUrl, autoLoad : true }, eventStore : { readUrl, autoLoad : true } }` (the Grid mapping above; tested at the Core tier 8.0.1: both stores request and load). The response is an `AjaxStore` one, `{ data : [...] }`: with `{ rows : [...] }` the request succeeds and the store stays empty, so set `responseDataProperty : 'rows'` or reshape the server reply. | TEST |
| a backend with its own envelope (several collections, nested objects such as `state : { id, name }`) | keep `loadUrl` and reshape in `project.listeners.beforeLoadApply({ response })`: mutate `response` in place (`response.tasks = { rows }`, `response.resources = { rows }`, `response.assignments = { rows }`). A TaskBoard column id and the task's `status` must be the same type (stringify both) | TEST |
| writes to the app's own PHP/Java endpoints (flat bodies `{ customer_name : 'x' }`, a `contatos` form field) | `AjaxStore` writes `{ data : [...] }` (delete: `{ ids }`) and expects `{ data : [...] }` back. A backend that must stay unchanged can't use `createUrl`/`updateUrl`/`deleteUrl`: send the flat body yourself (`AjaxHelper.post(url, body, { parseJson : true })`), `store.add(serverRecord)` and then `record.clearChanges()` on that record (public API; it takes the record out of `store.added`: tested). For a custom wire protocol subclass `AjaxStore` and override `commit()` (map `store.changes` to the calls, write server ids back). Otherwise change the server and say so in the plan | TEST |
| failed write (`Ext.Ajax` `failure`, proxy `exception`) | `AjaxHelper` with `parseJson` **rejects with the `Response` itself** for a non-JSON error body (`e.statusText`), but **resolves with `ok === false`** for a JSON error body: `if (!res.ok) throw res;` after every call | TEST |
| Ext reject-on-failure (`store.rejectChanges`) | reload (`project.load()`); `revertChanges()` doesn't undo a recurring "this and following" edit. 8.0.1, until bryntum/bryntum#14251 is fixed: `eventStore.revertChanges()` throws `view.getTime is not a function` once a recurring event has an exception date (`addExceptionDate`). Bryntum already shows a "Data synchronization failed" dialog on `syncFail` | TEST |
| data file that is a JS object literal (comments, unquoted keys), not JSON | a checked-in `scripts/convert-data.mjs` that `vm.runInNewContext`s it and writes JSON (`fetch().json()` and `loadUrl` fail on it) | TEST |
| `Ext.data.proxy.LocalStorage`, `direct`, `jsonp` | unmapped — flag for a human (`api-mapping.md` §8) | — |

The Ext Scheduler/Gantt CrudManager request/response envelope is close to Bryntum's but **not guaranteed identical**.
Compare an actual server response with the CrudManager guide
(`https://bryntum.com/products/gantt/docs/guide/Gantt/data/crud_manager`) before assuming the backend can stay as is.
Set `validateResponse : true` during development to log format errors. (SRC)

Response shape used by the Scheduler guide: `{ "resources" : { "rows" : [...] }, "events" : { "rows" : [...] } }` (G-S).

## 8.0 data-layer changes that touch Ext ports

- **Columns add missing fields.** `autoAddField` is now `true` (Grid, Scheduler, Gantt): a column whose `field` the model
  doesn't declare gets the field added, so it displays, edits and **is sent in sync requests**. This replaces the
  7.x trap "setting an undeclared record field is never saved" for column-backed fields. Fields that no column shows still
  need a declared field. Turn it off with `columns : { autoAddField : false, data : [...] }`. In Gantt that object form
  also drops the automatic `name` column, so list `{ type : 'name' }` yourself.
- **Time zones.** Dates are kept as exact moments and only the presentation is converted. An Ext app that shifted
  dates itself, or passes numeric UTC offsets (`-120`), must switch to IANA ids (`Etc/GMT+2`, `Europe/Stockholm`) and stop
  mutating record dates. `TimeZoneHelper` is deprecated.
- **Project transport on every tier**, as above. Don't configure transport on both the project and a `crudManager`: it throws.

## Scheduler

- Needs a `resourceStore` and an `eventStore` (instances, configs, or the `project` loading both; 7.x: `crudManager`). Default models:
  `ResourceModel`, `EventModel`. (G-S)
- Events need `id`, `resourceId` (or `assignments` for multi-resource), `startDate`, and `endDate` or
  `duration` + `durationUnit`; plus `name`. (G-S)

Inline mapping on the stores (G-S):

```js
project : {                    // 8.0, every tier (7.x Scheduler: crudManager)
    autoLoad      : true,
    resourceStore : {
        fields : [
            { name : 'id',        dataSource : 'YourIdField' },
            { name : 'name',      dataSource : 'Name' },
            { name : 'eventColor', dataSource : 'Color' }   // renderer-painted colors → eventColor (SRC)
        ]
    },
    eventStore : { modelClass : MyEvent },
    loadUrl    : 'api/load',
    validateResponse : true   // dev only
}
```

To map fields in a model subclass instead, use the same `{ name, dataSource }` entries in `static fields` on an
`EventModel` / `ResourceModel` subclass, with `eventStore : { modelClass }`. (G-S)

## Scheduler Enterprise (the former Scheduler Pro)

Put `resources`/`events`/`assignments`/`dependencies`/`calendars` on a `project` (the data + scheduling engine hub),
not ad hoc stores, whenever the app has dependencies, assignments or calendars. See `products/scheduler-enterprise.md`.

## Gantt

- `project` is mandatory and is the data hub. It includes the CrudManager, so `transport`/`loadUrl`, `autoLoad`,
  `stm` go on the project. Share one `ProjectModel` between the Gantt and any `Timeline`. (G-G)
- Custom models: `project : { taskModelClass, calendarModelClass, dependencyModelClass, ... }`. (G-G)

### Field renames (G-G, corrected)

Most fields are PascalCase → camelCase (`Id` → `id`, `StartDate` → `startDate`, `PercentDone` → `percentDone`,
`ConstraintType` → `constraintType`, `ShowInTimeline` → `showInTimeline`, `PerUseCost` → `perUseCost`, …). The
exceptions:

| Ext Gantt | Bryntum Gantt |
|---|---|
| Dependency `From` / `To` | `fromTask` / `toTask` |
| Assignment `TaskId` / `ResourceId` | **`event` / `resource`** (the guide's table says `taskId`/`resourceId` — wrong; SRC `AssignmentModel.js`) |

### Structural changes (the data itself must change)

- **`StartDate` semantics.** Ext kept `StartDate` as authored (only pushed by dependencies, and only when something
  was edited). Bryntum schedules ASAP from `project.startDate`, so a plain rename piles every task without a constraint
  at the project start. Two options; ask which the team wants and say so in the report:
  - **Keep every stored date (preferred when the Ext app did not cascade along dependencies).** Write
    `manuallyScheduled : true` on leaf tasks (parents still roll up from their children) and set
    `project : { skipNonWorkingTimeInDurationWhenSchedulingManually : true }` so durations count working days as Ext did.
    Two test ports kept all 26 start dates this way (a human reviewer compared each task). Trade-off: dependencies are drawn
    but do not move the successor. Parent durations can still differ from the stored ones (one parent showed 27 days
    against 25 stored): list those.
  - **Let dependencies push.** `project : { autoSetConstraints : true }` (every `startDate` becomes a `startnoearlierthan`
    constraint), or write `constraintType` / `constraintDate` into the data. In a test port this moved 13 of 26 tasks 1-8
    days later (weekends and dependency chains) and the parent ended about a week later; reviewers rated it an issue.
    Side effect: dragging a task then writes a constraint. Use it only when the team wants the engine to enforce
    dependencies, and diff the dates as described next.
- **A hand-posted write must handle failure.** When you write to a backend yourself (flat bodies, `AjaxHelper.post`): do
  not `record.set()` before the `await` unless the `catch` calls `record.revertChanges()`; wrap every awaited create, update
  and delete in try/catch, show the error the original showed, and reload or restore the row. A delete that fails silently
  or an edit that stays on screen after a rejected save is a behavior change you must report.
- **Dependency-violating source data changes on load.** Ext trusts stored dates, the Bryntum engine always recalculates
  (13 of 26 tasks moved in one app: chained finish-to-start tasks that were all the same day in the data). Dump
  start/end from the original and from `project.taskStore`, diff them, and list the tasks that moved as a behavior
  difference. Tasks without dependencies should match exactly: that is the proof the calendar conversion is right.
- **`TaskType`**: remove only the `Project` value (the project is no longer a task node). An app-defined `TaskType` such as
  `"Important"` / `"LowPrio"` is data: keep it as a custom field (`TaskModel` subclass, `static fields = ['taskType']`,
  `project.taskModelClass`).
- **Visible root row** (`rootVisible : true`): a Bryntum root is not a task. Drop it (the project name goes into
  `project.name`) and report the row-count difference (Ext 27 rows incl. root, Bryntum 26), or make it a real parent task.
- **`leaf`**: remove for Gantt. A node is a parent only if it has `children`. A TreeGrid loaded `leaf : true, children :
  null` rows fine (`children : null` is a leaf): the rule is Gantt-specific.
- **Project metadata**: new top-level `"project"` section, e.g.
  `{ "calendar" : "general", "startDate" : "2017-01-16", "hoursPerDay" : 24, "daysPerWeek" : 5, "daysPerMonth" : 20 }`.
- **Calendars**: only `id`, `name`, `intervals`. `Days` + `DefaultAvailability` → `intervals`:
  ```json
  { "id" : "general", "name" : "General", "intervals" : [
      { "recurrentStartDate" : "on Sat", "recurrentEndDate" : "on Mon", "isWorking" : false },
      { "name" : "Some big holiday", "startDate" : "2017-02-01", "endDate" : "2017-02-02", "isWorking" : false, "cls" : "holiday" }
  ] }
  ```
  `hoursPerDay`/`daysPerWeek`/`daysPerMonth` move from the calendar to the project (per-calendar escape hatch in
  `products/gantt.md`). Ext `BusinessTime` (Mon–Fri 08:00–12:00 and 13:00–17:00, 8 h/day, 5 d/week, 20 d/month) became
  `project : { calendar : 'general', hoursPerDay : 8, daysPerWeek : 5, daysPerMonth : 20 }` plus non-working recurring gaps
  (checked: durations and end dates identical to Ext for all tasks without dependencies):
  ```json
  { "recurrentStartDate" : "on Sat at 0:00",            "recurrentEndDate" : "on Mon at 0:00",           "isWorking" : false },
  { "recurrentStartDate" : "every weekday at 12:00",    "recurrentEndDate" : "every weekday at 13:00",   "isWorking" : false },
  { "recurrentStartDate" : "every weekday at 17:00",    "recurrentEndDate" : "every weekday at 08:00",   "isWorking" : false },
  { "name" : "Holiday", "startDate" : "2010-01-12", "endDate" : "2010-01-13", "isWorking" : false, "cls" : "holiday" }
  ```
  i.e. `DefaultAvailability` windows become the non-working gaps between them, and a single-date holiday `Date` gets the
  next day as `endDate`. The recurrence syntax has no `24:00` (`22:00–24:00` becomes `22:00–23:59`) and rejects
  `23:59:59`.
- **Baselines**: `BaselineStartDate` / `BaselineEndDate` → `"baselines" : [ { "startDate" : ..., "endDate" : ... } ]`.
  Columns, "set baseline" and sizing: `products/gantt.md`, "Baselines".

With a live backend, these structural changes mean either a server-side change or a load/sync adapter (a
server-side mapping layer, or client-side request/response hooks on the project: `beforeLoadApply({ response })` is
confirmed, see the stores table above; verify other hook names in the docs for the installed version). Record which one in
the plan; don't hide it. A demo backend that only echoes the posted body can't answer `project.sync()` (it needs
`{ requestId, success, tasks : { rows : [{ $PhantomId, id }] } }`): replace it with a CrudManager-protocol server and
say so (tested: phantom ids were replaced after Save).

### Scheduling modes (G-G)

| Ext Gantt | Bryntum Gantt |
|---|---|
| `Normal` / `FixedDuration` | same |
| `EffortDriven` | `FixedEffort` |
| `DynamicAssignment` | `FixedDuration` + `effortDriven : true` |
| — | `FixedUnits` (new) |

## Grid

Grid has no CrudManager: inline `data`, or a store with `readUrl` etc. (AjaxStore). Tree data: nested `children`
with a tree store (`tree : true`) and a `type : 'tree'` column. See `products/grid.md`.
