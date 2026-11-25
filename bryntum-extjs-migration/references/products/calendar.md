# Calendar (Ext `Cal.*`, Extensible and Ext JS `Ext.calendar`)

Support level: **limited**. No official Ext → Bryntum Calendar guide exists. The rows below were name-checked against the 8.0.0 `@bryntum/calendar-enterprise` package; rows tagged TEST were also run in a browser on 8.0.0
(real mouse input); "UNV" rows exist in Bryntum but the Ext equivalence is unverified. Report anything else as unmapped.

Tier: the Community tier has the base data model and the `day`, `week`, `month`, `year`, `agenda` views. **Recurring events need
the Core tier.** The Enterprise tier adds the `dayagenda`, `monthagenda`, `resource`, `dayresource`, `monthgrid`, `yearplanner` and
`multiweek` modes and `print`. Calendar keeps the Core tier data layer at the Enterprise tier. See `../api-mapping.md` §0.

Detect Bryntum's own Ext Calendar by `Cal.*` classes, `xtype : 'calendarpanel'` or `Cal.data.EventStore`. The **open-source Ext
calendar is Extensible** (`Extensible.calendar.CalendarPanel`, `xtype : 'extensible.calendarpanel'`,
`Extensible.calendar.data.EventMappings` / `CalendarMappings` / `MemoryEventStore`, Ext 3/4). **Ext JS's own calendar** is
`Ext.calendar.*` (Ext JS 6.2+, Classic and Modern: e.g. `Ext.calendar.panel.Panel`, `Ext.calendar.store.Calendars`, xtypes
like `calendar-month`). None of the `Cal.*` strings match either: use "Source: Extensible" or "Source: Ext JS
`Ext.calendar`" below, and ignore the `Cal.*` rows that don't apply (`eventRenderer(event, resource, metaData)`,
`Cal.panel.Calendar`). **Whatever the source, apply "Month and 2 Weeks: match the source" below**: the two calendars
differ there, and Bryntum's defaults match neither on its own.

| Ext (`Cal.*`) | Bryntum Calendar | Tag |
|---|---|---|
| `Cal.panel.Calendar` | `new Calendar({ appendTo, ... })` | UNV |
| `date`, `mode` (`day`/`week`/`month`/`year`/`agenda`), `readOnly`, `weekStartDay`, `hideNonWorkingDays`, `nonWorkingDays`, `showWeekNumber` | same names | SRC (`weekStartDay`, `hideNonWorkingDays`) |
| `eventStore`, `resourceStore`, CrudManager | `project : { eventStore, resourceStore, loadUrl, syncUrl, autoLoad }` (8.0: the project is the CrudManager) | SRC |
| `eventEditor : {...}` | `features : { eventEdit : {...} }` (vanilla; wrappers use `eventEditFeature`, see `SKILL.md`, Converting) | SRC, TEST |
| `eventMenu`, `dayMenu` | `features : { eventMenu, scheduleMenu }`, both on by default. `scheduleMenu` is the day-cell menu (right-click an empty cell; one built-in item `addEvent`). There is no `dayMenu` feature: `features : { dayMenu }` throws `Invalid type name`. Customize with `features : { scheduleMenu : { items : { addEvent : false, myItem : { text, onItem } } } }` (`false` removes a default item) and the Calendar listeners `scheduleMenuBeforeShow` (return `false` to suppress the menu), `scheduleMenuShow`, `scheduleMenuItem({ item, date })` (`item.ref` is the item key). Same in week and month view | TEST |
| `recurringEvents : true` | drop the config: it isn't a Calendar config (stored as an inert property). Recurrence is built in: an event with `recurrenceRule : 'FREQ=DAILY;COUNT=3'` renders 3 occurrences with no config (Core tier and up) | TEST |
| `navigationToolbar`, `resourceFilter` | `tbar`, or the built-in `sidebar : { items : { resourceFilter : {...} } }` | SRC |
| `modes : { day, week, month, year, agenda }` | `modes : { week : {...}, ... }`, set a mode to `null` to remove it | SRC |
| `eventRenderer(event, resource, metaData)` | `eventRenderer({ view, eventRecord, resourceRecord, renderData })`, returning a `DomConfig`, string or nothing. `renderData` keys: `style`, `cls`, `iconCls`, `eventColor`, `eventBackground`, `textColor`, `solidBar`, … (not Ext's `metaData`). A top-level `eventRenderer` on the Calendar is **silently ignored**: use `modeDefaults : { eventRenderer }` for every view, or `modes : { week : { eventRenderer } }` for one. `renderData.cls.x = true` adds a class | TEST |
| `dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn` | one feature `drag` (`CalendarDrag`): `features : { drag : { validateMoveFn, validateResizeFn, validateCreateFn } }`, each `({ drag, event, view, eventRecord }) => boolean` (create may return a `ValidateCreateResult`). Returning `false` blocks it. `eventDrag`/`eventResize`/`eventDragCreate` don't exist on Calendar (`new Calendar` throws `Invalid type name`). Grab the middle of an event to move it: near the top edge you get the resize handle | TEST |

Rules:

- Calendar has **no dependencies** and no time axis. Don't carry Scheduler rows over.
- New views that Ext didn't have (`list`, `resource`, `dayresource`, `dayagenda`, `monthagenda`) are off by default: don't
  enable them, but suggest the ones that fit in the report. **`year` and `agenda` are on by default**: a source with
  Day/Week/Month/2 Weeks needs `year : null, agenda : null` in `modes` to show the same four buttons (tested).
- Data: events need `startDate` + `endDate` or `duration`; `allDay : true` for all-day events. PascalCase fields go through
  `dataSource` mapping as in `../data-migration.md`.
- **All-day end dates.** Ext/Extensible all-day events end on the same day (inclusive, or `23:59:59`). Bryntum's `endDate`
  is exclusive: set `startDate` to midnight and `endDate` to midnight of the day after the last day, or the event is
  zero-length. Clear stray times on all-day seed data (some start at 01:00).
- **Editor.** Calendar's editor is flat. `widgetMap` keys: `nameField`, `resourceField`, `allDay`, `startDateField`,
  `startTimeField`, `endDateField`, `endTimeField`, `colorField`, `recurrenceCombo`, `editRecurrenceButton`. A custom item
  needs `weight` above 800 (built-ins use 100–800), or it lands between "End date" and "End time". `items : { resourceField
  : false }` removes the always-present, empty "Calendar" combo of an app without calendars. `eventEdit : { triggerEvent :
  'eventclick' }` reproduces an Ext single-click edit window (default: double-click).
- **Multi-week view.** `modes : { multiweek : { range : '2 w', title : '2 Weeks' } }` (Enterprise tier). `weekCount` is a
  getter: setting it throws `Cannot set property weekCount ... which has only a getter`. The default button text is
  "Multi week": `title` renames it.
- **Sidebar.** Extensible's "Jump to" field and `CalendarListPanel` map to the built-in sidebar (a date picker and a resource
  filter list with colour check boxes, sorted by name, not by store order): no config needed.
- A mode's events are `.b-cal-event-wrap`; occurrence ids look like `_generated:<masterId>:<yyyy-mm-dd>`. In a test scope
  month cells to `calendar.activeView.element` (the sidebar date picker uses the same `.b-calendar-cell` class).
  `window.bryntum.fromElement(el)` returns the widget.

## Month and 2 Weeks: match the source

These come from the source **library's** views, not the app's code, so the app's files never mention them: identify the
source (above) and set the Bryntum app to match, in both `month` and `multiweek`.

| Behaviour | Extensible | Ext JS `Ext.calendar` | Bryntum default | Set |
|---|---|---|---|---|
| Timed event (not all-day) text | **start time before the title**: `8:00am Haircut` (`'g:ia '`; `'G:i '` with `Extensible.Date.use24HourTime`), from `src/calendar/view/Month.js` `getTemplateEventData`; multi-day timed bars too | **start time before the title**: `07:00 Bike Ride` (seen as `H:i`; use the app's time format if it sets one) | title only (`showTime` and the event header are DayView/WeekView only) | the time renderer below, Bryntum format `'h:mma'` for Extensible, `'HH:mm'` for `H:i` |
| "+N more" popup lists | **all** the day's events (`MonthDayDetail`) | **only the hidden** ones: "+7 more" opens 7, not the events already in the cell | all the day's events | Extensible: nothing. `Ext.calendar`: `overflowPopup : { onlyShowOverflow : true }` |

```javascript
const TIME_FORMAT = 'h:mma';   // Extensible 'g:ia' → 8:00am; Ext.calendar 'H:i' → 'HH:mm'

// Month and 2 Weeks only: Day/Week keep their built-in time header
function timedRenderer({ view, eventRecord, renderData }) {
    const name = StringHelper.encodeHtml(eventRecord.name);

    if (eventRecord.reminder) {           // the app's own extras (here Extensible's reminder bell) go here too:
        renderData.iconCls = 'fa fa-bell'; // a mode's eventRenderer replaces the modeDefaults one
    }
    return eventRecord.allDay ? name : view.dateHelper.format(eventRecord.startDate, TIME_FORMAT) + ' ' + name;
}

modes : {
    month     : { eventRenderer : timedRenderer /* Ext.calendar: , overflowPopup : { onlyShowOverflow : true } */ },
    multiweek : { eventRenderer : timedRenderer, range : '2 w', title : '2 Weeks' /* same overflowPopup */ }
}
```

Tested on 8.0.2 (Extensible app): Month and 2 Weeks show `9:30am Team stand-up`, each recurring occurrence with its own start
time, all-day bars and Week view unchanged, and the "+N more" popup uses the same renderer. "+4 more" opened 7 events by
default and the 4 hidden ones with `onlyShowOverflow` (month; `multiweek` takes the same config, untested). `'HH:mm'` is
untested. Compare each in the browser against the original (or a screenshot of it) before reporting it done.

## Source: Extensible

Tested on `examples/calendar/basic` (vanilla) and `remote/recurrence` (React): event counts matched, a real-mouse drag and
resize gave the same result as the original, and occurrences for the same month matched.

| Extensible | Bryntum |
|---|---|
| `EventMappings` `EventId`, `Title`, `StartDate`, `EndDate`, `IsAllDay`, `CalendarId`, `RRule` | `id`, `name`, `startDate`, `endDate`, `allDay`, `resourceId`, `recurrenceRule` |
| `Location`, `Notes`, `Url`, `Reminder` | no built-ins: `EventModel` subclass with `static fields`, set as `eventModelClass` on the project (React: in the `project` prop) |
| `Duration`, `OriginalEventId`, `RSeriesStartDate`, `RInstanceStartDate`, `REditMode` | drop: server-side expansion plumbing |
| `CalendarModel` `CalendarId`, `Title`, `Description`, `ColorId`, `IsHidden` | `ResourceModel` `id`, `name`, custom field, `eventColor`; `IsHidden` has no field (an initial resource filter) |
| colour id (`ColorId`, 1–32) | the hex in `resources/css/calendar-colors.css` (`.x-cal-<id>-x dl { background-color }`): `eventColor : '#cf2424'` |
| custom `{ name, mapping, dateFormat }` mappings (the custom-mappings example) | a `dataSource` mapping, or convert in the loader |
| Reminder | **no equivalent**: keep a `reminder` string field, a `combo` item with `ReminderCombo.getValueList()`, and a bell through `modeDefaults : { eventRenderer({ eventRecord, renderData }) { if (eventRecord.reminder) renderData.iconCls = 'fa fa-bell'; return eventRecord.name; } }`. That renderer drops the start time in Month and 2 Weeks: give those two modes the time renderer in "Month and 2 Weeks: match the source", with the same bell |
| Month and MultiWeek: start time before timed events' titles; "+N more" lists all the day's events | see "Month and 2 Weeks: match the source" |
| `Extensible.calendar.view.MultiWeek` | `multiweek` mode (above) |
| locales `extensible-lang-*.js` | already shipped: `import '@bryntum/calendar-<tier>/locales/calendar.locale.De.js'; LocaleManager.applyLocale('De')`. Names: `ca`→`Ca`, `zh_CN`→`ZhCn`, `zh_TW`→`ZhTw`, `hr`→`Hr`, `cs`→`Cs`, `da`→`Da`, `nl`→`Nl`, `en_GB`→`EnGb`, `fr`→`FrFr`, `de`→`De`, `it`→`It`, `pl`→`Pl`, `pt_BR`→`PtBr`, `pt_PT`→`Pt`, `ro`→`Ro`, `es`→`Es`, `sv_SE`→`SvSE`. `LocaleManager` comes from `@bryntum/core-<tier>` |

**Recurrence: server-expanded (Extensible) vs client-expanded (Bryntum).** Extensible's server expands each RRULE for the
requested range, returns instances with ids `<id>-rid-<n>`, stores the series' end in the row's `end`, a `Duration` in
minutes and an exceptions table, and receives edits with `redit = single | future | all`, `origid`, `ristart`. Bryntum loads
**master rows only**: `recurrenceRule`, `endDate` = the first occurrence's end, `exceptionDates` (`['2026-11-20T08:00:00']`
works, time of day included). A conversion script is needed, and the server's expansion/exception logic is deleted, not
ported: it only persists masters. Edits arrive as plain CrudManager changes:

- "Only This Event" → the master's `exceptionDates` is updated and a **new standalone event** (`recurrenceRule : null`) is added
- "All Future Events" → the master's rule gets `;UNTIL=<yyyyMMddTHHmmss>` (no `Z`) and a **new event** carries the full rule
- delete one occurrence → `exceptionDates`
- dates in sync payloads are ISO **with offset** (`2026-11-23T08:00:00+05:00`) while loaded data had none: the server must accept both; a phantom id is replaced through `$PhantomId`

Behavior differences to report: Extensible's Recurring Event Options window has Single / Future / **All Events** (default
All); Bryntum's popup has "Only This Event" / "All Future Events" / Cancel, no "all" (edit the first occurrence). The editor's
repeat legend can read "Monthly on 27" for `FREQ=MONTHLY;BYDAY=-1FR` while the saved rule stays intact. Ext's PHP capped a
series at 99 instances.

Backend: Ext reject-on-failure is a reload in Bryntum (`syncFail({ source }) { setTimeout(() => source.load(), 0); }`):
see `data-migration.md` (Ext reject-on-failure) for the `revertChanges()` limits and the 8.0.1 throw.

Running the original: `Extensible-config.js` points to a dead CDN, `extensible-all-debug.js` and `extensible-all.css` are
build outputs missing from the clone, and the PHP demo needs MySQL. What worked: Ext 4.2.1 from another checkout, a Node
static server that prepends `window.ExtensibleDefaults = { extJsRoot : '/ext/', mode : 'dynamic' }` to
`Extensible-config.js` and serves `calendar-colors.css` + `calendar.css` + `recurrence.css` as `extensible-all.css`, and a Node
reimplementation of the PHP API.

## Source: Ext JS `Ext.calendar`

Not yet tested against an `Ext.calendar` app: only its Month view behaviour was checked (against a screenshot). Apply "Month
and 2 Weeks: match the source" above (start time before titles, `onlyShowOverflow : true`). The Bryntum side of the `Cal.*`
rows (modes, `eventEdit`, `drag`, menus, `eventRenderer` placement) and the Rules apply. Data (`Ext.calendar.store.Calendars`,
each calendar with its own events store, and the event model's fields) has no verified mapping: map it field by field as
in `../data-migration.md`, mark those rows UNV in the plan, and compare the views in a browser with the original.
