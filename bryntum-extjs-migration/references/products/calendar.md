# Calendar (Ext `Cal.*`)

Support level: **limited**. No official Ext → Bryntum Calendar guide exists. The rows below come from the suite's Ext
migration knowledge base and were name-checked against the 8.0.0 `@bryntum/calendar-enterprise` package; rows tagged TEST were also run in a browser on 8.0.0
(real mouse input); "UNV" rows exist in Bryntum but the Ext equivalence is unverified. Report anything else as unmapped.

Tier: the Community tier has the base data model and the `day`, `week`, `month`, `year`, `agenda` views. **Recurring events need
the Core tier.** The Enterprise tier adds the `dayagenda`, `monthagenda`, `resource`, `dayresource`, `monthgrid`, `yearplanner` and
`multiweek` modes and `print`. Calendar keeps the Core tier data layer at the Enterprise tier. See `../api-mapping.md` §0.

Detect it by `Cal.*` classes, `xtype : 'calendarpanel'` or `Cal.data.EventStore`.

| Ext (`Cal.*`) | Bryntum Calendar | Tag |
|---|---|---|
| `Cal.panel.Calendar` | `new Calendar({ appendTo, ... })` | UNV |
| `date`, `mode` (`day`/`week`/`month`/`year`/`agenda`), `readOnly`, `weekStartDay`, `hideNonWorkingDays`, `nonWorkingDays`, `showWeekNumber` | same names | SRC (`weekStartDay`, `hideNonWorkingDays`) |
| `eventStore`, `resourceStore`, CrudManager | `project : { eventStore, resourceStore, loadUrl, syncUrl, autoLoad }` (8.0: the project is the CrudManager) | SRC |
| `eventEditor : {...}` | `features : { eventEdit : {...} }` | SRC |
| `eventMenu`, `dayMenu` | `features : { eventMenu, scheduleMenu }`, both on by default. `scheduleMenu` is the day-cell menu (right-click an empty cell; one built-in item `addEvent`). There is no `dayMenu` feature: `features : { dayMenu }` throws `Invalid type name`. Customize with `features : { scheduleMenu : { items : { addEvent : false, myItem : { text, onItem } } } }` (`false` removes a default item) and the Calendar listeners `scheduleMenuBeforeShow` (return `false` to suppress the menu), `scheduleMenuShow`, `scheduleMenuItem({ item, date })` (`item.ref` is the item key). Same in week and month view | TEST |
| `recurringEvents : true` | drop the config: it isn't a Calendar config (stored as an inert property). Recurrence is built in: an event with `recurrenceRule : 'FREQ=DAILY;COUNT=3'` renders 3 occurrences with no config (Core tier and up) | TEST |
| `navigationToolbar`, `resourceFilter` | `tbar`, or the built-in `sidebar : { items : { resourceFilter : {...} } }` | SRC |
| `modes : { day, week, month, year, agenda }` | `modes : { week : {...}, ... }`, set a mode to `null` to remove it | SRC |
| `eventRenderer(event, resource, metaData)` | `eventRenderer({ view, eventRecord, resourceRecord, renderData })`, returning a `DomConfig`, string or nothing. `renderData` keys: `style`, `cls`, `iconCls`, `eventColor`, `eventBackground`, `textColor`, `solidBar`, … (not Ext's `metaData`). A top-level `eventRenderer` on the Calendar is **silently ignored**: use `modeDefaults : { eventRenderer }` for every view, or `modes : { week : { eventRenderer } }` for one. `renderData.cls.x = true` adds a class | TEST |
| `dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn` | one feature `drag` (`CalendarDrag`): `features : { drag : { validateMoveFn, validateResizeFn, validateCreateFn } }`, each `({ drag, event, view, eventRecord }) => boolean` (create may return a `ValidateCreateResult`). Returning `false` blocks it. `eventDrag`/`eventResize`/`eventDragCreate` don't exist on Calendar (`new Calendar` throws `Invalid type name`). Grab the middle of an event to move it: near the top edge you get the resize handle | TEST |

Rules:

- Calendar has **no dependencies** and no time axis. Don't carry Scheduler rows over.
- New views that Ext didn't have (`list`, `resource`, `dayresource`, `dayagenda`, `monthagenda`) are off by default: don't
  enable them, but suggest the ones that fit in the report.
- Data: events need `startDate` + `endDate` or `duration`; `allDay : true` for all-day events. PascalCase fields go through
  `dataSource` mapping as in `../data-migration.md`.
