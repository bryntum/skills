# Gantt

Support level: **strong** (an official Ext → Bryntum Gantt guide exists; its mappings are corrected and folded into
`../api-mapping.md` §4–§6, `../data-migration.md`, `../patterns.md`).

Finished example (extjs-migration-agent repo): `gantt-extjsmodern-vite` (Ext hbox + resizable panel → `Container` +
`Splitter`, Ext grouped List → Bryntum `List`, wrapper defaults moved onto the Gantt, `ProjectModel` + dataset copy,
Material3, RTL, fixing a default overwritten by `items : { x : true }`).

## What a Bryntum Gantt app looks like

- `new Gantt({ appendTo, project, columns, features, tbar })`; `project` is mandatory (data + scheduling hub)
- Gantt `startDate`/`endDate` = visible time axis; `project.startDate` = project start
- Gantt-native columns (`wbs`, `name`, `startdate`, `duration`, `predecessor`, `addnew`, ...)
- features: `baselines`, `dependencies`, `dependencyEdit`, `rollups`, `progressLine`, `criticalPaths`, `rowReorder`,
  `timeRanges`, `fillHandle`, `cellCopyPaste`, `taskCopyPaste`, ...
- toolbars are `Toolbar` subclasses passed as `tbar : { type : '<name>' }`

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
`.b-row-number-cell` and `layout : 'vbox'`. Two non-obvious details:

- Live animation duration needs a `<style>` node rewritten on each slider input — `transitionDuration` alone doesn't
  change the CSS transition:
  ```js
  construct(...args) {
      super.construct(...args);
      this.styleNode = document.createElement('style');
      document.head.appendChild(this.styleNode);
  }
  onAnimationDurationChange({ value }) {
      this.gantt.transitionDuration = value;
      this.styleNode.innerHTML = `.b-animating .b-gantt-task-wrap { transition-duration: ${value / 1000}s !important; }`;
  }
  ```
- Bar-margin slider max depends on row height: `barMargin.max = (rowHeight.value / 2) - 5`.

### Per-calendar duration conversion (rare)

Ext Gantt calendars had their own `hoursPerDay`/`daysPerWeek`/`daysPerMonth`; Bryntum moved these to the project. Only
if the app really depends on per-calendar values:

```js
class MyCalendarModel extends DurationConverterMixin.derive(CalendarModel) {}
// project : { calendarModelClass : MyCalendarModel, taskModelClass : MyTaskModel, dependencyModelClass : MyDependencyModel }
```

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

### Old zipped copies of the Ext Gantt demos

Don't carry over `@bryntum/gantt ^5.6.12`, combined theme CSS (`gantt.stockholm.css`), a top-level `pan : true`
(belongs in `features`), or debug `console.log` lines.

## Both types supported

- **Type A** (`Gnt.*`, PascalCase data, `TaskType`, old calendars): the full mapping in `../api-mapping.md` and the
  structural data changes in `../data-migration.md`.
- **Type B** (Bryntum Gantt in an Ext wrapper): use `gantt-extjsmodern-vite` as the template.

No finished type A example exists yet (a legacy `Gnt.*` fixture has been prepared for one).
