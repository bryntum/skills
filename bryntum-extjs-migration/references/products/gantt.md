# Gantt

Support level: **strong** (an official Ext → Bryntum Gantt guide exists; its mappings are corrected and folded into
`../api-mapping.md` §4–§6, `../data-migration.md`, `../patterns.md`).

**Tier:** an Ext Gantt maps to Gantt at the **Core** tier or above when it used constraints, calendars/working time, effort,
lag/lead or dependencies (Gantt Community runs a lighter engine without them: report them as a gap if the user keeps the
Community tier), and to the **Enterprise** tier when it used baselines, critical path, `projectEdit`, rollups, summary,
PDF export or print (`../api-mapping.md` §0).

`project` is mandatory and is the data + scheduling hub. Gantt `startDate`/`endDate` set the visible time axis, and
`project.startDate` is the project start.

## Rules

### Baselines

`BaselineStartDate`/`BaselineEndDate` → `baselines : [{ startDate, endDate, cls }]` (Enterprise tier). On 8.0.1:

- columns: `{ type : 'baselinestartdate' }`, `{ type : 'baselineenddate' }` (field `baselines[0].startDate`), and for a second
  baseline `{ type : 'baselinestartdate', text : 'Secondary Start Date', field : 'baselines[1].startDate' }`
- an extra `cls` in the data (`cls : 'secondary-baseline'`) lands on `.b-task-baseline`: show/hide toggles are plain CSS on the
  Gantt (`.b-gantt .b-task-baseline { display : none }`), like Ext's `sch-ganttpanel-showbaseline`
- "set new baseline" (Ext: `node.set('BaselineStartDate', node.getStartDate())` per node) → `task.setBaseline(1)` (1-based:
  copies start/end into version 1). **Don't** assign `task.baselines = [{...}, ...old.data]`: records without an id are
  appended and the list reorders
- `task.baselines.getAt(0)` is typed `Model`, so `.startDate` is a TypeScript error: read a version with `task.getBaseline(1)` (typed `Baseline`) or cast
- baselines shrink the bar even when hidden with `display : none`: one port also set `--b-baseline-wrap-size : 0` on the
  Gantt while baselines were hidden, and the bar returned to full height
- baselines shrink the bar: the bar is `rowHeight − 2 × barMargin` and baselines take 0.6 of it. `rowHeight : 30` with
  `barMargin : 6` gave 7 px bars, and the percent-bar handle then swallowed every drag (`percentBar : { allowResize : false }`,
  `barMargin : 2` gave 10 px)
- differences to report: the `baselineenddate` column shows the stored **exclusive** end (Feb 1) where Ext showed the
  inclusive day (Jan 31); Ext snapped baseline starts on non-working days to the previous working day; a secondary baseline
  stacks one lane lower instead of overlaying

### Resource views on the same project

`ResourceHistogram` and a resource `Scheduler` share the Gantt's `project` and `partner : gantt` (`api-mapping.md` §1; set
`weekStartDay` on each: Sunday by default). `Scheduler` needs `columns` or no resource names show. In Angular `[partner]`
takes `ganttCmp.instance`. **The scale has an equivalent**: Ext `scaleMin`/`scaleMax`/`scaleStep` → the `scale` column's `scalePoints`
(`columns : [{ type : 'scale', scalePoints : [{ value : 0, unit : 'hour' }, { value : 4, unit : 'hour', text : '4' }, { value : 8, unit : 'hour', text : '8' }] }]`;
the views read the unit from the last point). The default is 8h/16h/24h, so a histogram you leave alone runs 0–24 h and a full 8-hour working day
fills a third of the row (all 3 E4 runs; the Ext original was 0–8 h). Read from `ScaleColumn.js` in 8.0.x; check the bar heights against Ext
in the browser. (A Resource Histogram drew no bars in
one early test with 8.0.1 and `getRecordHistogramData` returning data; not reproduced in a minimal case and unfiled: unverified, don't promise it.)

### Color settings from `Ext.util.CSS.updateRule`

On 8.0.1: `gantt.eventColor = x` colors all bars (the percent bar derives from it). Variables set on `.b-gantt` do **not**
recolor bars: `--b-gantt-task-primary` and `--b-primary` are redefined on the task wrappers, and `--b-percent-bar-background`
needs a rule on the bar (`.b-gantt .b-task-percent-bar { background : var(--app-progress-color) }`). `--b-dependency-color` on `.b-gantt`
works. A `ColorField` returns palette names (`b-red`): pass them through `DomHelper.createColorStyle()` before using them
in CSS. For hex values give the field a `colors` palette and compare uppercase.

### Timeline companion

An overview strip above/below the Gantt is a `Timeline` sharing the Gantt's `project` — not a second Gantt. On 8.0 `Timeline` is exported by `@bryntum/scheduler-<tier>` (declare it as a dependency), `ProjectModel` by the Gantt package:

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
if the app really depends on per-calendar values (`DurationConverterMixin` ships in the engine package: `import { DurationConverterMixin } from '@bryntum/engine-<tier>/lib/Engine/scheduling/DurationConverterMixin.js'`; it is not in `gantt.d.ts`), follow "Restoring calendar level duration converting" in the Gantt
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
  structural data changes in `../data-migration.md`. Check Gantt toolbar items too: a `'->'` spacer is
  `{ type : 'widget', flex : 1 }` (a string item in keyed `items` is a type error), `fullscreenbutton` isn't a registered type
  (use a button calling `element.requestFullscreen()`), and `onToggle` on a `toggleable` button gives `{ pressed }`.
- **Type B** (Bryntum Gantt in an Ext wrapper): remove the Ext shell (an Ext hbox + resizable panel becomes a
  `Container` + `splitter`), move the wrapper's defaults onto the Gantt, and re-check the config against 8.0 defaults.
