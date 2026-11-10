# Calendar (Ext `Cal.*`)

Support level: **limited**. No official Ext → Bryntum Calendar guide exists. The rows below come from the suite's Ext
migration knowledge base and were name-checked against the 8.0.0 `@bryntum/calendar-enterprise` package; "UNV" rows
exist in Bryntum but the Ext equivalence is unverified. Report anything else as unmapped.

Tier: Community has the base data model and the `day`, `week`, `month`, `year`, `agenda` views. **Recurring events need
Core.** Enterprise adds the `dayagenda`, `monthagenda`, `resource`, `dayresource`, `monthgrid`, `yearplanner` and
`multiweek` modes and `print`. Calendar keeps the Core data layer at Enterprise. See `../api-mapping.md` §0.

Detect it by `Cal.*` classes, `xtype : 'calendarpanel'` or `Cal.data.EventStore`.

| Ext (`Cal.*`) | Bryntum Calendar | Tag |
|---|---|---|
| `Cal.panel.Calendar` | `new Calendar({ appendTo, ... })` | UNV |
| `date`, `mode` (`day`/`week`/`month`/`year`/`agenda`), `readOnly`, `weekStartDay`, `hideNonWorkingDays`, `nonWorkingDays`, `showWeekNumber` | same names | SRC (`weekStartDay`, `hideNonWorkingDays`) |
| `eventStore`, `resourceStore`, CrudManager | `project : { eventStore, resourceStore, loadUrl, syncUrl, autoLoad }` (8.0: the project is the CrudManager) | SRC |
| `eventEditor : {...}` | `features : { eventEdit : {...} }` | SRC |
| `eventMenu`, `dayMenu` | `features : { eventMenu, scheduleMenu }` | SRC (names), UNV (`dayMenu` equivalence) |
| `recurringEvents : true` | built in (Core tier and up) | UNV |
| `navigationToolbar`, `resourceFilter` | `tbar`, or the built-in `sidebar : { items : { resourceFilter : {...} } }` | SRC |
| `modes : { day, week, month, year, agenda }` | `modes : { week : {...}, ... }`, set a mode to `null` to remove it | SRC |
| `eventRenderer(event, resource, metaData)` | `eventRenderer({ eventRecord, resourceRecord, renderData })` | UNV |
| `dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn` | `validatorFn` of `eventDrag` (Calendar's drag feature), `eventResize`, `eventDragCreate` | UNV: verify the argument shape |

Rules:

- Calendar has **no dependencies** and no time axis. Don't carry Scheduler rows over.
- New views that Ext didn't have (`list`, `resource`, `dayresource`, `dayagenda`, `monthagenda`) are off by default: don't
  enable them, but suggest the ones that fit in the report.
- Data: events need `startDate` + `endDate` or `duration`; `allDay : true` for all-day events. PascalCase fields go through
  `dataSource` mapping as in `../data-migration.md`.
