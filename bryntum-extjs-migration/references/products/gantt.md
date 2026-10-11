# Gantt

Support level: **strong** (an official Ext → Bryntum Gantt guide exists; its mappings are corrected and folded into
`../api-mapping.md` §4–§6, `../data-migration.md`, `../patterns.md`).

`project` is mandatory and is the data + scheduling hub. Gantt `startDate`/`endDate` set the visible time axis, and
`project.startDate` is the project start.

## Rules

### Timeline companion

An overview strip above/below the Gantt is a `Timeline` sharing the Gantt's `project` — not a second Gantt:

```js
const project  = new ProjectModel({ transport : { load : { url : 'api/load' } }, autoLoad : true });
const timeline = new Timeline({ appendTo : 'app', project, height : '12em' });
const gantt    = new Gantt({ appendTo : 'app', project /* , ... */ });
```

### Full-featured toolbar

For a toolbar with zoom, undo/redo, filters and a live settings menu (row height, bar margin, animation duration,
dependency radius), model it on the official `advanced` demo's `GanttToolbar`
(`https://bryntum.com/products/gantt/examples/advanced/`) rather than reconstructing it. It uses `onAction`,
`.b-row-number-cell` and `layout : 'vbox'`. A bar-margin slider's max depends on row height:
`barMargin.max = (rowHeight.value / 2) - 5`.

### Per-calendar duration conversion (rare)

Ext Gantt calendars had their own `hoursPerDay`/`daysPerWeek`/`daysPerMonth`; Bryntum moved these to the project. Only
if the app really depends on per-calendar values, follow "Restoring calendar level duration converting" in the Gantt
calendars guide (`https://bryntum.com/products/gantt/docs/guide/Gantt/basics/calendars`). It takes three pieces, and
all are required:

- `MyCalendarModel extends DurationConverterMixin.derive(CalendarModel)` — this only **adds** the
  `hoursPerDay`/`daysPerWeek`/`daysPerMonth` fields to calendars
- `MyTaskModel` overriding `convertDurationGen` (convert with the task's `effectiveCalendar`) and `canConvertDuration`
- `MyDependencyModel` overriding `convertLagGen` (convert with the dependency's calendar)

Then `project : { calendarModelClass, taskModelClass, dependencyModelClass }`. With the mixin alone (empty task and
dependency subclasses), conversion silently keeps using the project values. Copy the overrides from the guide.

### Language switcher

Build the combo from the registered locales (inside a widget, e.g. the toolbar, where `me = this`):

```js
const locales = Object.keys(me.localeManager.locales).map(key => ({
    value : key, text : me.localeManager.locales[key].localeDesc
}));
me.widgetMap.localeCombo.store.data = locales;
me.widgetMap.localeCombo.value      = me.localeManager.locale.localeName;
// on change: me.localeManager.applyLocale(value);
```

## Both types supported

- **Type A** (`Gnt.*`, PascalCase data, `TaskType`, old calendars): the full mapping in `../api-mapping.md` and the
  structural data changes in `../data-migration.md`.
- **Type B** (Bryntum Gantt in an Ext wrapper): remove the Ext shell (an Ext hbox + resizable panel becomes a
  `Container` + `splitter`), move the wrapper's defaults onto the Gantt, and re-check the config against 7.x defaults.
