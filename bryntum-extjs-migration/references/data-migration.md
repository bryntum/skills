# Data migration: models, stores, loading, saving

Tags as in `api-mapping.md`. For CrudManager/AjaxStore mechanics (phantom IDs, partial sync, common data gotchas), also
load the `bryntum-crud` skill.

## Pick one strategy per app and record it in MIGRATION_PLAN.md

1. **Map fields in the model** — the server and its JSON stay unchanged; Bryntum fields read from the old names via
   `dataSource`. Default when a real backend exists that the team doesn't want to change. (G-S, G-G)
2. **Convert the data** to Bryntum's camelCase fields and structures. Required for structures that can't be expressed
   as field mappings (Gantt calendars, `TaskType`, baselines). For static JSON, convert with a checked-in script
   (e.g. `scripts/convert-data.mjs`) so it can be re-run. For a real backend, this means a server change — say so in
   the plan and get the user's agreement. (G-G)

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
| field `calculate` / `convert` | `calculate : r => ...` for derived fields (SRC); `convert` — verify (UNV) | SRC / UNV |
| `idProperty : 'Id'` | `idField : 'Id'` (store) or `{ name : 'id', dataSource : 'Id' }` | G-S |
| `hasMany` / `belongsTo` associations | no direct equivalent — flatten, or use Bryntum's built-in relations (assignments, dependencies, tree children) | UNV |
| Sch/Gnt model subclasses (`Sch.model.Event`, `Gnt.model.Task`) | `EventModel`, `ResourceModel`, `TaskModel`, `DependencyModel`, `AssignmentModel`, `CalendarModel` | G-S, G-G |

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
| `sorters`, `groupers`, `filters` | same names on the store. A 7.x `grouper.field` must be a field name (a function throws); use a calculated field for computed groups | SRC |
| `store.sync()` | AjaxStore `commit()`; CrudManager `sync()` | DOC |
| Sch/Gnt `CrudManager` (`transport`, `load`, `sync`) | Bryntum `crudManager` (Scheduler) or `project` (Scheduler Pro, Gantt) with `loadUrl`/`syncUrl` or `transport` | G-S, G-G |
| separate event/resource stores loaded by separate proxies (Scheduler) | either keep separate stores with `readUrl`, or one `crudManager` load for both | SRC |
| `Ext.data.proxy.LocalStorage`, `direct`, `jsonp` | unmapped — flag for a human (`api-mapping.md` §8) | — |

The Ext Scheduler/Gantt CrudManager request/response envelope is close to Bryntum's but **not guaranteed identical**.
Compare an actual server response with the CrudManager guide
(`https://bryntum.com/products/gantt/docs/guide/Gantt/data/crud_manager`) before assuming the backend can stay as is.
Set `validateResponse : true` during development to log format errors. (SRC)

Response shape used by the Scheduler guide: `{ "resources" : { "rows" : [...] }, "events" : { "rows" : [...] } }` (G-S).

## Scheduler

- Needs a `resourceStore` and an `eventStore` (instances, configs, or a `crudManager` loading both). Default models:
  `ResourceModel`, `EventModel`. (G-S)
- Events need `id`, `resourceId` (or `assignments` for multi-resource), `startDate`, and `endDate` or
  `duration` + `durationUnit`; plus `name`. (G-S)

Inline mapping on the stores (G-S):

```js
crudManager : {
    autoLoad      : true,
    resourceStore : {
        idField : 'YourIdField',
        fields  : [
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

Model subclass mapping (G-S):

```js
import { EventModel } from '@bryntum/scheduler';

export default class MyEvent extends EventModel {
    static $name = 'MyEvent';
    static fields = [
        { name : 'name',       type : 'string', dataSource : 'Title' },
        { name : 'resourceId', dataSource : 'ResourceId' },
        { name : 'startDate',  type : 'date',   dataSource : 'StartDate' },
        { name : 'endDate',    type : 'date',   dataSource : 'EndDate' },
        { name : 'location',   dataSource : 'Location' }
    ];
}
```

## Scheduler Pro

Put `resources`/`events`/`assignments`/`dependencies`/`calendars` on a `project` (the data + scheduling engine hub),
not ad hoc stores, whenever the app has dependencies, assignments or calendars. See `products/schedulerpro.md`.

## Gantt

- `project` is mandatory and is the data hub. It includes the CrudManager, so `transport`/`loadUrl`, `autoLoad`,
  `stm` go on the project. Share one `ProjectModel` between the Gantt and any `Timeline`. (G-G)
- Custom models: `project : { taskModelClass, calendarModelClass, dependencyModelClass, ... }`. (G-G)

### Field renames (G-G, corrected)

| Ext Gantt | Bryntum Gantt |
|---|---|
| `Id`, `Name`, `Description`, `Cls` | `id`, `name`, `description`, `cls` |
| `StartDate` / `EndDate` / `Duration` / `PercentDone` | `startDate` / `endDate` / `duration` / `percentDone` |
| `Rollup`, `Segments`, `Resizable`, `Draggable` | `rollup`, `segments`, `resizable`, `draggable` |
| `AllowDependencies`, `ShowInTimeline` | `allowDependencies`, `showInTimeline` |
| `ConstraintType` / `ConstraintDate` | `constraintType` / `constraintDate` |
| `Rate` / `PerUseCost` / `Units` | `rate` / `perUseCost` / `units` |
| Dependency `From` / `To` | `fromTask` / `toTask` |
| Assignment `TaskId` / `ResourceId` | **`event` / `resource`** (the guide's table says `taskId`/`resourceId` — wrong; SRC `AssignmentModel.js`) |
| General rule | PascalCase → camelCase |

### Structural changes (the data itself must change)

- **`TaskType`**: remove. The project is no longer a task node.
- **`leaf`**: remove. A node is a parent only if it has `children`.
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
  `products/gantt.md`).
- **Baselines**: `BaselineStartDate` / `BaselineEndDate` → `"baselines" : [ { "startDate" : ..., "endDate" : ... } ]`.

With a live backend, these structural changes mean either a server-side change or a load/sync adapter (a
server-side mapping layer, or client-side request/response hooks on the CrudManager — verify the hook names in the
docs for the installed version). Record which one in the plan; don't hide it.

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
