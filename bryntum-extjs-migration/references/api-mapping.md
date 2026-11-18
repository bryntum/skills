# API mapping: Ext JS → Bryntum 8

Source names checked against the Bryntum **8.0.0** packages (7.3.7 for rows that say so; 7.x differences are in
`v7.md`). For a newer installed version, confirm anything tagged other than SRC with the MCP
`search_bryntum_docs` tool (pass the installed `version`) or the docs.

Tags:

- **G-S** / **G-G** — stated in the official Ext → Bryntum Scheduler / Gantt migration guide (corrected where wrong)
- **SRC** — confirmed in the 8.0 / 7.3.7 source (check it in `node_modules/@bryntum/<product>-<tier>/` for the installed version)
- **DOC** — added for general Ext app migrations; the name is confirmed in the 7.3.7/8.0 typings (`*.d.ts`), but the
  mapping hasn't been verified end to end — check config details in the docs before relying on them
- **TEST** — also run in a browser on 8.0.0 with real mouse input (Calendar and TaskBoard rows)
- **UNV** — the Bryntum name exists, but equivalence to the Ext behavior is not verified

Product-specific rows only apply to that product. Don't generalize a Gantt row to Scheduler or vice versa.

## 0. Tiers: what each Ext feature needs on 8.0

The tiers are the **Community < Core < Enterprise** tiers; install `-core`, never the deprecated `-business` (see the base
skill's "Tiers"). `@bryntum/core-core` is the Core package at the Core tier, not a typo.

A feature above the installed tier is skipped, and a dialog names the tier it needs. Check each ported plugin before you
promise it. **Rule:** a feature is available at a tier if that tier's packages contain it. Use the table below, or check
the installed packages.

| Product | Core tier and up | Enterprise tier only |
|---|---|---|
| Grid | `filter` (header-menu filter), `group`, `cellCopyPaste`, `rowCopyPaste`, `cellTooltip`, `columnDragToolbar`, `columnRename`, `pinColumns`, `stickyCells`, `treeGroup` | `summary`, `groupSummary`, `rowExpander`, `rowReorder`, `lockRows`, `split`, `mergeCells`, `charts`, `pdfExport`, `print`, `rowEdit`, `fillHandle`, `rowResize`, AI |
| Scheduler | `dependencyEdit`, `dependencyMenu`, `eventCopyPaste`, `eventDragSelect`, `eventFilter`, `headerZoom`, `labels`, `nonWorkingTime`, `pan`, `percentBar`, `resourceColumnReorder`, `resourceTimeRanges`, `scheduleContext`, `scrollButtons`, `simpleEventEdit`, `stickyEvents`, `timeRanges`, `timeSelection` | `taskEdit`, `eventClustering`, `resourceTimeRangeClustering`, `eventNonWorkingTime`, `resourceNonWorkingTime`, `summary`, `timelineSummary`, `treeSummary`, `groupSummary`, `tickCells`, `lockRows`, `rowReorder`, `split`, `pdfExport`, `print`, `eventBuffer`, `eventSegments` (+ drag/resize), `nestedEvents`, `groupRollups`, `resourceEdit`, `calendarHighlight`, `timeSpanHighlight`, `allocationCellEdit`, `allocationCopyPaste`, `rowResize`, `versions`, AI |
| Gantt | `taskEdit`, `taskCopyPaste`, `taskNonWorkingTime`, `progressLine`, `projectLines`, `indicators`, `labels`, `parentArea`, `scrollButtons`, `treeGroup`, `rowExpander`, `rowReorder` (on by default) | `summary`, `pdfExport`, `print`, `baselines`, `criticalPaths`, `projectEdit`, `rollups`, `timelineChart`, `taskSegmentDrag`/`taskSegmentResize`, MS Project export, `versions`, AI; `cost` column (`CostColumn`) |
| Calendar | recurring events (data layer), `eventCopyPaste`, `externalEventSource`, `timeRanges`, `weekExpander` | `eventBuffer`, `loadOnDemand`, `print`, AI; views `dayagenda`, `monthagenda`, `resource`, `dayresource`, `monthgrid`, `yearplanner`, `multiweek` |
| TaskBoard | swimlanes, `columnFilter`, `columnRename`, `columnResize`, `filterBar`, `simpleTaskEdit`; card items `jsx`, `resourceAvatars`, `tags`, `todoList` | `columnDrag`, `columnLock`, `columnSort`, `swimlaneDrag`, card items `chart`, `collapse`, AI |

A Gantt port reads the Grid and Scheduler rows too (`fillHandle`, `groupSummary`, `rowResize`, `resourceEdit` are listed there, not under Gantt).
`ResourceGrid` (the resource view with cost/rate columns, in `scheduler-<tier>`) is Enterprise only, so a Core-tier port loses it.
`ResourceHistogram` and `ResourceUtilization` ship in `scheduler-<tier>` and are exported by the Gantt package: they need the
Enterprise tier for a Scheduler and the Core tier for a Gantt.
Run at the Core tier on a Scheduler (tested): `filterBar`, `filter`, `timeRanges`, `resourceTimeRanges`, `workingTime`,
the built-in presets, `zoomIn`, `dependencies` (`dependencyEdit` errors without it), `labels`, `percentBar`, `headerZoom`,
`pan`, `scrollButtons`, `scheduleContext`, `timeSelection`, `simpleEventEdit`, `eventDragSelect`. Confirmed Enterprise-only by the
"Features unavailable on this tier" dialog: `rowReorder`, `print`, `pdfExport`, `summary`, `excelExporter`, `eventBuffer`,
`eventClustering`, `eventSegments`, `nestedEvents`, `eventNonWorkingTime`, `resourceNonWorkingTime`, `resourceEdit`,
`calendarHighlight`, `timeSpanHighlight`. `tickCells` fails with a factory error (no dialog) at Core. Three import
traps at Core: `type : 'action'` columns throw `Invalid type name "action"` because the Scheduler entry doesn't register
`ActionColumn` (`import '@bryntum/grid-<tier>/lib/Grid/column/ActionColumn.js'` (here `grid-core`), or import Grid's package); `features : { tree }`
(a `Sch.panel.SchedulerTree`) throws `Invalid type name "tree" passed to SchedulerFeature factory` until
`import '@bryntum/grid-<tier>/lib/Grid/feature/Tree.js'` is in the app (tested at Core 8.0.1 with a tree `resourceStore`: it then
works; `Tree.js` is `@tier community`, so this is not a tier limit); and a wrapper prop for
a feature the tier lacks (`taskEditFeature`) is a `tsc` error while the vanilla `features : { taskEdit : false }` is silently
ignored.

In Gantt, `rowExpander` and `rowReorder` are Core tier on purpose (the Enterprise label on them applies to Grid and Scheduler only),
and `rowReorder` is on by default, so don't report an Ext `gridviewdragdrop` or `rowexpander` on a Core tier Gantt as a gap or turn it off.

Engine and data layer:

- **Gantt Community** runs a lighter engine (base task: dates, name, `percentDone`, WBS): no working time, constraints,
  critical path, lag/lead, effort or scheduling direction. **Gantt Core** has the full engine (constraints,
  dependencies, calendars). **Gantt Enterprise** adds baselines and project revisions.
- **Calendar Community** has no recurring events (the Core tier adds them, with lazy loading): an Ext Calendar with recurrence needs the Core tier.
- **Scheduler Core** adds recurring events and lazy loading to the plain data model. **Scheduler Enterprise** swaps in
  the scheduling engine (constraints, effort, `percentDone`, calculated dates) and the default editor becomes `taskEdit`
  (`taskEdit` ships only in the Enterprise package; the Community and Core tiers use `eventEdit`). It is the former Scheduler Pro.
  A Scheduler that used neither the engine nor an Enterprise-only feature ran identically at the Core tier in tests; the tier
  is still the user's choice (`SKILL.md`, "Before starting").
- Models and stores resolve to the tier's class from the plain name (`TaskModel` → `TaskModelEnterprise`). To run the
  Enterprise package at a lower tier set `tier : 'core'` on the widget. Don't `instanceof` across tiers: use the `is<Name>` flags.
- A field a lower tier doesn't have reads as `undefined`, silently. Re-check Ext data fields such as `Effort`,
  `ConstraintType` and `Baselines` against the tier.
- Re-check a feature in the installed packages: it is there if `node_modules/@bryntum/*-<tier>/lib/*/feature/<Name>.js` exists in any package of the tier. A Gantt app's Grid and Scheduler features ship in `grid-<tier>` and `scheduler-<tier>`, and a Scheduler app's Grid features in `grid-<tier>`, not under the product's own `lib/`. Don't go by `static tier`.

## 1. Classes and application structure

| Ext JS | Bryntum 8 | Notes | Tag |
|---|---|---|---|
| `Sch.panel.SchedulerGrid` / `Sch.panel.SchedulerPanel` | `Scheduler` | `new Scheduler({ appendTo, ... })`. With dependencies, assignments or calendars: Enterprise tier (7.x: `SchedulerPro`, see `v7.md`) | G-S |
| `Gnt.panel.Gantt` | `Gantt` | | G-G |
| `Ext.grid.Panel` / `Ext.grid.Grid` (Modern) | `Grid` | | DOC |
| `Ext.tree.Panel` | `TreeGrid` (or `Grid` with a tree store + `type : 'tree'` column) | | DOC |
| `Gnt.panel.ResourceHistogram` (`scalecolumn`, `scaleStep`/`scaleMin`/`scaleMax`) | `ResourceHistogram` with `project` + `partner : gantt`, `columns : [{ field : 'name', flex : 1 }, { type : 'scale', width : 70 }]`, `rowHeight`, `hideHeaders`, `showBarTip`. The Ext scale configs have no equivalent (the scale max comes from the time axis): report as unmapped | SRC |
| `Sch.panel.SchedulerGrid` showing the Gantt's tasks and resources (`eventStore : taskStore`, `partnerTimelinePanel`) | `Scheduler` with the Gantt's `project` + `partner : gantt` (Enterprise tier); give it `columns : [{ text : 'Name', field : 'name', flex : 1 }]`, or no resource names show | SRC |
| `Gnt.model.Project` node (via `TaskType`) | `ProjectModel` | Not a task node: a wrapper holding all stores + CrudManager + scheduling engine | G-G |
| Scheduling done by `Gnt.data.TaskStore` | `ProjectModel` | `await project.commitAsync()` after programmatic changes | G-G |
| `Ext.application`, controllers, `Ext.Viewport` | none | Plain ES module (vanilla) or the framework's app shell. Widgets use `appendTo` | G-G |
| `Ext.define('X', { extend : 'Y' })` | `class X extends Y {}` | Widgets: `static type = 'x'`, `static $name = 'X'`, then `X.initClass()` to register | G-G, SRC |
| `Ext.create('X', cfg)` | `new X(cfg)` or `Widget.create(cfg)` | | G-G |
| `xtype` | `type` | | G-G |
| `ptype` / `ftype` plugins | `features : { name : true \| {config} }` | Toggle at runtime via `feature.disabled` | G-G |
| `Ext.panel.Panel` / `Ext.container.Container` | `Panel` / `Container` | Use `Panel` for panels with features (header, tools); use `Container` for pure layout wrappers | SRC |
| `Ext.tab.Panel` | `TabPanel` | | DOC |
| `Ext.Viewport` / hbox panel | `new Panel({ appendTo, layout : 'hbox' })` | Viewport becomes a Panel when migrating; pure layout containers use `Container` with `layout` | SRC |
| panel `resizable : { split : true }` / `split : true` | `{ type : 'splitter' }` item between panels | | SRC |
| `Ext.Toolbar` / `dockedItems` / `tbar` | `tbar` / `bbar` on the widget, or a `Toolbar` subclass | Keyed `items : { myButton : {...} }`, access via `widgetMap` | G-G, SRC |
| panel `header : { items }` | Panel `tools` | Any widget config | SRC |
| `Ext.toolbar.Paging` | `bbar : { type : 'pagingtoolbar' }` + AjaxStore remote paging | | DOC |
| `Ext.menu.Menu` | `Menu` | | DOC |
| `Ext.tip.ToolTip` / `data-qtip` | `Tooltip` / `data-btip` | Grid cell tooltips (renderer `metaData.tdAttr = 'data-qtip=…'`): column `tooltipRenderer` + `features : { cellTooltip : true }`, which is off by default | SRC |
| `Ext.window.Toast` / `Ext.toast(msg)` | `Toast.show(msg)` | | SRC |
| `Ext.Msg.alert(title, msg, fn)` | `await MessageDialog.alert({ title, message })` | Single OK button. `message` is rendered as HTML, so escape user data | SRC |
| `Ext.Msg.confirm(title, msg, fn)` | `await MessageDialog.confirm({ title, message, okButton : 'Yes', cancelButton : 'No' }) === MessageDialog.okButton` | Returns a `Promise<number>`. `message` is rendered as HTML, so escape user data | SRC |
| `Ext.Msg.prompt` | `MessageDialog.prompt({ title, message, textField })` | Resolves to `{ button, text }` | SRC |
| `Ext.Dialog` / `Ext.window.Window` + form | `Popup` subclass (`modal`, `centered`, `closable`, `autoShow : false`, `autoClose : false`, `bbar` buttons, `keyMap`). In a framework app with its own component system, use its dialog | See `patterns.md` §6 | SRC |
| `xtype : 'list'` + `itemTpl`, `grouped`, store `grouper` | `type : 'list'`, `itemTpl(record)`, `groupHeaderTpl(record, groupName)`, `collapsibleGroups`, store `groupers` | | SRC |
| overrides of Ext classes (`Ext.define(null, { override : 'Ext.field.Select' })`) | drop | They patch Ext bugs. List them in the report | SRC |

## 2. Form fields

| Ext `xtype` | Bryntum `type` | Tag |
|---|---|---|
| `textfield` | `textfield` (or `text`) | SRC |
| `textareafield` | `textareafield` | DOC |
| `numberfield` / `spinnerfield` (`minValue`, `maxValue`) | `numberfield` (`min`, `max`) | SRC |
| `combobox` / `combo` / `selectfield` | `combo` (`items` array or `store`, `valueField`, `displayField`) | SRC |
| `datefield` | `datefield` | SRC |
| `timefield` | `timefield` (`step : '30min'`) | SRC |
| `checkbox` / `checkboxfield` | `checkbox` | DOC |
| `radiogroup` | `radiogroup` (`options : { value : 'Label' }`) | SRC |
| `displayfield` | `displayfield` | DOC |
| `fieldcontainer` / `formpanel` | `Container` / `Popup` with `layout` | SRC |
| `allowBlank : false` | `required : true` | SRC |
| `regex` / `regexText`, `vtype`, custom `validator` | none direct. A `checkValidity` config on the field works: `checkValidity(field) { re.test(field.value) ? field.clearError(MSG, true) : field.setError(MSG, true); }`. **The second argument `true` (silent) is required**: without it `clearError` re-enters `checkValidity` and overflows the stack. `maskRe` (blocks keystrokes) has no equivalent | TEST |
| `maxLength`, `emptyText` | `maxLength`, `placeholder` | SRC |
| `timeConfig : { minValue : '08:00', maxValue : '18:00' }` | **not** `timefield : { min : '08:00' }`: the string is parsed as 2020-01-01 08:00 and is not enforced in the event editor (07:00 stayed valid). Set Date `min`/`max` on the event's own day in `beforeEventEditShow` (`DateHelper.add(DateHelper.clearTime(eventRecord.startDate), 8, 'hour')`): with Dates 07:00 became invalid and 09:00 stayed valid (tested at Core 8.0.1). The 12h display default also differs: `format : 'HH:mm'` | TEST |
| `iconCls : 'x-fa fa-sync'` (button, tool, column) | `icon : 'fa fa-sync'` (widgets), `renderData.iconCls : 'fa fa-sync'` (events). The `b-fa b-fa-*` classes log "Found legacy .b-fa CSS class" on 8.0: use unscoped `fa` | TEST |
| `form.loadRecord`, `trackResetOnLoad`, `dirtychange`, `isDirty`, `reset`, `getValues` | `patterns.md` §6 | TEST |

## 3. Scheduler-specific configs

| Ext Scheduler | Bryntum Scheduler | Tag |
|---|---|---|
| `title` | keep — Scheduler is a Panel (`title`, `tools`, `tbar` work). The guide's "remove" is outdated | SRC |
| `border`, `colorResources` | remove | G-S |
| `rowHeight`, `forceFit`, `viewPreset` | keep | G-S |
| `snapToIncrement` | `snap` | G-S, SRC |
| `createEventOnDblClick` | remove (default behavior) | G-S |
| `lockedGridConfig : { width }` | `subGridConfigs : { locked : { width } }` | G-S, SRC |
| — | `barMargin` (defaults differ, set explicitly) | G-S |
| `eventBodyTemplate` (Ext.XTemplate) | `eventRenderer` returning a DomConfig (preferred), or an HTML string with record values escaped via the `StringHelper.xss` tagged template / `StringHelper.encodeHtml()` — a direct `XTemplate` port can open an XSS hole | G-S, SRC |
| `eventRenderer(event, resource, tplData)` | `eventRenderer({ eventRecord, resourceRecord, renderData })` | SRC |
| `tooltipTpl` | `features : { eventTooltip : { template : ({ eventRecord }) => '...' } }` | G-S |
| `setViewPreset(p)` / `switchViewPreset(p, start, end)` | `scheduler.viewPreset = p` + `setTimeSpan(start, end)`, or `scheduler.zoomTo({ preset, startDate, endDate })`. Either way the axis snaps to whole units of the preset: `weekAndDay` starts weeks on `weekStartDay` (Sunday by default), so a Mon–Mon span becomes two weeks. Align the start with `DateHelper.startOf(date, 'week')` or set `weekStartDay : 1` | G-S, SRC |
| `setTimeSpan(s, e)` | `setTimeSpan(s, e)` | SRC |
| `scheduler_zones` / `Sch.plugin.Zones` (store of `Sch.model.Range`) | project `timeRanges` with `startDate`, `endDate`, `name`, `cls` + `features : { timeRanges : { showHeaderElements : true } }`. DOM: `.b-sch-time-range.b-sch-range.<cls>`. Add at runtime with `scheduler.timeRangeStore.add({...})` | TEST |
| `scheduler_lines` / `Sch.plugin.Lines` (`Date`, `Text`, `Cls`) | a `timeRanges` entry **without** `endDate` or duration is drawn as a line (`.b-sch-time-range.b-sch-line.<cls>`; `name` is its label) | TEST |
| `scheduler_currenttimeline` (`updateInterval` ms) | `features : { timeRanges : { showCurrentTimeLine : { updateInterval : <seconds> } } }` | TEST |
| `scheduler_printable` | `features : { print }` (Enterprise tier) | SRC |
| custom `Sch.data.TimeAxis` hiding hours (`continuous : false`, `generateTicks`) | `workingTime : { fromDay, toDay, fromHour, toHour }`. A drop outside the hidden hours becomes impossible, so Ext "outside working hours" confirms can't fire | TEST |
| `zoomLevels` | `presets : ['weekAndMonth', 'weekAndDay', 'hourAndDay', ...]` (string ids accepted; zoom is limited to them). Without it, a "min zoom" jumps to `manyYears` | TEST |
| built-in preset date formats (`hourAndDay` shows `8:00` in Ext) | Bryntum's `hourAndDay` shows `8 AM`. Pass `viewPreset : { base : 'hourAndDay', headers : [{ unit : 'day', dateFormat : 'ddd DD/MM' }, { unit : 'hour', dateFormat : 'H:mm' }] }` | TEST |
| `eventBarIconClsField` | `renderData.iconCls = eventRecord.<field>`: it renders `<i class="Meeting">` without `b-icon`, so an old `.Meeting { background : url() }` rule needs `.b-sch-event i.Meeting` plus a width and height | TEST |
| `allowOverlap`, `eventBarTextField`, `readOnly`, `zoomOnMouseWheel` | same names | UNV |
| `mode : 'vertical'` / `orientation` | `mode : 'vertical'` | UNV |
| `dndValidatorFn` | `features.eventDrag.validatorFn` (one context object, same shape as below) | UNV |
| `constrainDragToResource` | not a Scheduler config (TypeScript error): it is an `eventDrag` option, false by default | TEST |
| `beforeeventdrop` listener returning `false` | `beforeEventDropFinalize` (there is no `beforeEventDrop`, checked in 8.0.0): set `context.valid = false`, or `context.async = true` + `context.finalize(bool)` | SRC |
| `resizeValidatorFn(resource, event, start, end, e)` | `features.eventResize.validatorFn({ startDate, endDate, eventRecord, ... }, event)` returning `{ valid : false, message }`: the message shows in the resize tooltip and the resize reverts | TEST |
| `createValidatorFn` | `features.eventDragCreate.validatorFn` | UNV |
| `multiSelect` (events) | `multiEventSelect` | UNV |
| `resourceImagePath` | `resourceImagePath` or `resourceImages : { path, extension }` | SRC |
| event colors (renderer-injected) | `eventColor` field/config + `eventStyle` (`'tonal'`, `'filled'`, `'bordered'`, `'traced'`, `'outlined'`, `'indented'`, `'line'`, `'dashed'`, `'minimal'`, `'rounded'`). `eventColor` takes any CSS color but does not derive the text color: for light custom colors set `color` in `renderData.style` or use a palette name | SRC, TEST |
| `StartDate` as epoch milliseconds (`dateFormat : 'time'`) | a `date` field does **not** parse numbers (silently `null`, nothing renders), and a `convert` on `startDate`/`endDate` breaks drag-create at the Enterprise tier (`getTime is not a function`) and crashes the project on load at the Core tier. Convert to `Date`/ISO before loading (a loader or script) | TEST |

## 4. Gantt-specific configs

| Ext Gantt | Bryntum Gantt | Tag |
|---|---|---|
| `showTodayLine` | `features.timeRanges.showCurrentTimeLine : true` | G-G |
| `loadMask` | `loadMask` | G-G |
| `enableProgressBarResize : false` (the Ext default) | `percentBar : { allowResize : false }`. Bryntum's percent-bar drag is on by default and, on a short bar, takes the drag a task move needed (3 of 3 task drags failed). `true` → leave the default | G-G, TEST |
| `enableDependencyDragDrop : false` | `features : { dependencies : { allowCreate : false } }` (no terminals appear on hover) | TEST |
| `showRollupTasks` | `features.rollups` | G-G, SRC |
| `rowHeight`, `viewPreset` | keep | G-G |
| — | `barMargin` (default differs a lot) | G-G |
| `projectLinesConfig` | `features.projectLines` (no target config) | G-G, SRC |
| `allowDeselect` | remove (CTRL-click always deselects) | G-G |
| `selModel : { type : 'gantt_spreadsheet' }` | none — see §8 | G-G |
| `lockedGridConfig : { width }` | `subGridConfigs : { locked : { width } }` (or `flex`) | G-G |
| `lockedViewConfig.getRowClass` | `cls` field on the record, or a column `renderer` | G-G |
| `taskBodyTemplate` | `taskRenderer` | G-G, SRC |
| `eventRenderer(task, tplData)` | `taskRenderer({ taskRecord, renderData })` | G-G, SRC |
| `leftLabelField : { dataIndex, editor }` | `features.labels.left : { field, editor : { type : 'textfield' } }` | G-G |
| `startDate` (view) | Gantt `startDate` = time-axis start; `project.startDate` = project start | G-G |
| `zoomIn()` / `zoomOut()`, `switchViewPreset('year', a, b)` | same names, but the neighbouring presets differ (`weekAndDayLetter` zooms in to `sixHoursAndDay`, Ext to `weekAndDay`) and the axis snaps to whole units (`zoomTo({ preset, startDate, endDate })`). Weeks start on Sunday: set `weekStartDay : 1` on the Gantt and on every Scheduler/histogram beside it | TEST |
| baseline `BaselineStartDate`/`EndDate`, "set baseline" | `products/gantt.md`, "Baselines" | TEST |

## 5. Plugins / grid features → features

| Ext | Bryntum feature | Tag |
|---|---|---|
| `cellediting` / `scheduler_treecellediting` (`clicksToEdit`) | `cellEdit` (default on in Grid-based widgets). `clicksToEdit : 1` → `cellEdit : { triggerEvent : 'cellclick' }` (default is double-click). A grid **without** the `cellediting` plugin isn't editable in Ext even if its columns have `field`: set `cellEdit : false` | TEST |
| `rowediting` | `rowEdit` (Enterprise tier; compare its UX with the Ext row editor) | DOC |
| `gridfilters` | `filter` (header menu) or `filterBar` (filter row) | G-G, SRC |
| `ftype : 'grouping'` | `group` feature (`features : { group : 'field' }`). `hideGroupedHeader` has no Group config: set `hidden : true` on the grouped column (`hideGroupedColumns` exists only on `treeGroup`) | SRC |
| `ftype : 'summary'` | `summary` feature + column `sum` (Enterprise tier, §0) | DOC |
| `ftype : 'groupingsummary'` | `groupSummary` (Enterprise tier) | DOC |
| `rowexpander` / `ftype : 'rowbody'` | `rowExpander` (Grid: Enterprise tier, §0) | DOC |
| `bufferedrenderer` | remove — virtual rendering is built in | DOC |
| `clipboard` / `gantt_clipboard` | `cellCopyPaste` (+ `taskCopyPaste : { useNativeClipboard : true }` in Gantt) | G-G, SRC |
| `gridviewdragdrop` (reorder rows) | `rowReorder` (Enterprise tier in Grid and Scheduler; Core tier and on by default in Gantt). **On a sorted store it clears the sorters and the drop is lost.** Keep the order in a field, sort the data on load, and renumber the field in `gridRowDrop` (`store.forEach((r, i) => r.order = i + 1)`) | G-G, TEST |
| `treeviewdragdrop` (re-parent, or between two trees with a shared `ddGroup`) | `rowReorder : { dropOnLeaf : true, allowCrossGridDrag : { transferData({ records, insertBefore, parent, sourceGrid, targetGrid }) {} } }`: a drop onto a leaf makes it a parent. Without `transferData` the node is moved (copied with Ctrl). Tested between two TreeGrids with a real mouse | TEST |
| drag onto a timeline (from a list or a Grid) | `DragHelper`: see the `bryntum-drag-and-drop` skill. Its demo takes the start date from the proxy's left edge, Ext used the pointer date (`getDateFromDomEvent(e, 'round')`). A chained store (`eventStore.chain(fn)`) re-evaluates only on fields passed as the 2nd argument: `chain(fn, ['startDate'])`, or the dragged row stays in the list | TEST |
| context menus (`itemcontextmenu` listener, `advanced_taskcontextmenu`, `scheduler_eventcontextmenu`) | `cellMenu`, `headerMenu` (Grid); `eventMenu` (Scheduler); `taskMenu` (Gantt) — on by default. Scheduler `eventMenu` item keys at the Enterprise tier: `editEvent`, `showDetails`, `copyEvent`, `cutEvent`, `splitEvent`, `eventColor`, `unassignEvent`, `deleteEvent` (the table is at the top of `EventMenu.js`); `editEvent : false` does not remove "Show details". `deleteEvent : { text }` renders; `deleteEvent : true` doesn't (SKILL.md trap) | G-G, SRC, TEST |
| `scheduler_pan` | `pan`. In a **Gantt** it throws `Invalid type name "pan" passed to GanttFeature factory` until `import '@bryntum/scheduler-<tier>/lib/Scheduler/feature/Pan.js'` is in the app (declare the dependency) | G-G, SRC, TEST |
| Scheduler event editor plugin | `eventEdit`, customize with keyed `items` + `weight` | SRC |
| `gantt_taskeditor` (`taskFormClass`) | `taskEdit`, customize `items : { generalTab : { items : {...} } }` | G-G |
| `gantt_projecteditor` | `projectEdit` (Enterprise tier; the guide says obsolete) | SRC |
| `gantt_dependencyeditor` | `dependencyEdit` | G-G, SRC |
| `gantt_progressline` | `progressLine : { statusDate, disabled }` | G-G |
| drag-drop / resize / drag-create | `eventDrag`, `eventResize`, `eventDragCreate` (default on) | SRC |
| dependencies | `dependencies : { radius, clickWidth, showLagInTooltip }` | G-G |
| critical path / baselines / non-working time | `criticalPaths` / `baselines` / `nonWorkingTime` | G-G |
| row resize | `rowResize : { cellSelector : '.b-row-number-cell' }` | G-G (corrected) |
| `gantt_selectionreplicator` | none; `fillHandle` is closest (behavior differs) | G-G, UNV |
| demo plugin `taskarea` | none (`parentArea` may be close) | UNV |

## 6. Columns

General (all Grid-based widgets):

| Ext | Bryntum | Tag |
|---|---|---|
| `header` / `text` | `text` | G-S |
| `dataIndex` | `field` | G-S |
| `editor` / `field` (editor config) | `editor` (text is default; omit when plain text); `editor : false` for read-only | G-S |
| `flex`, `width`, `hidden`, `align` | same names | G-S |
| `locked : true` | `locked : true` (or `region`) | SRC |
| `sortable`, `menuDisabled`, `filter` | `sortable`; `filterable` (with `filterBar`); header menu per `headerMenu` | DOC |
| `renderer(value, meta, record)` | `renderer({ value, record, cellElement, row, ... })` — return the content. A bare formatter (`renderer : fmt`) got Ext's `meta` as its 2nd argument: check `fmt`'s optional params and pass what that did | SRC |
| `columns : [{ text, columns : [...] }]` (grouped headers) | `children : [...]` on a parent column; `collapsible` groups | SRC |
| `xtype : 'datecolumn'` (`format : 'Y-m-d'`) | `type : 'date'` (`format : 'YYYY-MM-DD'` — convert tokens) | DOC |
| `xtype : 'numbercolumn'` | `type : 'number'` | DOC |
| `xtype : 'checkcolumn'` / `booleancolumn` | `type : 'check'` | DOC |
| `xtype : 'templatecolumn'` (`tpl`) | `type : 'template'`, `template : ({ record }) => ...` | SRC |
| `xtype : 'actioncolumn'` (`items`, `handler`) | `type : 'action'`, `actions : [{ cls, tooltip, onClick }]` (tested). A scheduler column `position : 'right'` (right of the timeline) has no equivalent: it stays in the locked grid | TEST |
| `xtype : 'rownumberer'` | `type : 'rownumber'` | DOC |
| `xtype : 'treecolumn'` | `type : 'tree'` | DOC |
| `xtype : 'widgetcolumn'` | `type : 'widget'`, `widgets : [...]` | DOC |
| Ext `combobox` editor + `store` | `editor : { type : 'combo', items : [...] }` | G-S |

Gantt-only column types (G-G): `namecolumn` → `name`, `startdatecolumn` → `startdate`, `enddatecolumn` → `enddate`,
`durationcolumn` → `duration`, `constrainttypecolumn` → `constrainttype`, `constraintdatecolumn` → `constraintdate`,
`percentdonecolumn` → `percentdone`, `predecessorcolumn` → `predecessor`, `addnewcolumn` → `addnew`, `dragdropcolumn` →
remove (use `rowReorder`). New: `wbs`.

Scheduler resource column with avatar: `type : 'resourceInfo'` (SRC).

## 7. Selection, methods, events, helpers

| Ext JS | Bryntum | Tag |
|---|---|---|
| `selModel : { selType : 'checkboxmodel' }` | `selectionMode : { checkbox : true }` | DOC |
| `selModel : 'cellmodel'` | `selectionMode : { cell : true }` | DOC |
| `selModel.mode : 'MULTI'` | `selectionMode : { multiSelect : true }` | DOC |
| default Ext selection (`rowmodel`, one row) | `selectionMode : { multiSelect : false }`: Bryntum selects several rows with Ctrl/Shift-click by default | SRC |
| grid `select` / `selectionchange` events | `selectionChange` | SRC |
| `itemclick` / `itemdblclick` / `cellclick` | `cellClick` / `cellDblClick` (Grid); `eventClick` / `eventDblClick` (Scheduler); `taskClick` (Gantt) | DOC |
| `listeners : { x : fn, scope }` | `listeners : { x : fn, thisObj }` or `widget.on({ x })` (returns a detacher). A subclass method named `on<Event>` is already a listener (`callOnFunctions`), so don't also register it | G-G, SRC |
| button `handler` | `onClick` / `onAction`; string `'up.methodName'` calls the method on the nearest ancestor that has it, passing the click event. See `patterns.md` §3 | G-G, SRC |
| `Ext.getCmp(id)` / `lookupReference(ref)` / `down('#id')` | `widgetMap.ref`, `Widget.getById(id)` | G-G, SRC |
| `Ext.XTemplate` | function returning a template literal; escape data with `StringHelper.xss` tagged template | SRC |
| `Ext.Date.format(d, 'Y-m-d H:i')` | `DateHelper.format(d, 'YYYY-MM-DD HH:mm')` — **tokens differ** (PHP-style → moment-style: `Y`→`YYYY`, `m`→`MM`, `d`→`DD`, `H`→`HH`, `i`→`mm`, `s`→`ss`, `g:i A`→`h:mm A`, `D`→`ddd`, `l`→`dddd`, `M`→`MMM`, `F`→`MMMM`, `j`→`D`, `n`→`M`) | SRC |
| `Ext.Date.add(d, Ext.Date.DAY, 1)` | `DateHelper.add(d, 1, 'day')` | SRC |
| `Ext.Date.clearTime(d)` | `DateHelper.clearTime(d)` | SRC |
| `grid.setDisabled(true)` | `grid.disabled = 'inert'`. 8.0.1, until bryntum/bryntum#TBD is fixed: `true` only adds a `b-disabled` class (no greying) and a mouse click still selects a row | TEST |
| `suspendLayouts` / `resumeLayouts` | `suspendRefresh()` / `resumeRefresh()` | UNV |
| `expandAll`, `collapseAll`, `zoomIn`, `zoomOut`, `zoomToFit`, `shiftPrevious`, `shiftNext` | same names. Wrap them in a handler rather than using `'up.shiftNext'` (the event object would become `amount`) | G-G, SRC |

Store, record, tree-node and component-lookup methods (`store.first`, `findRecord`, `insertChild`, `down()`,
`getView().refresh()`, …) and the near-misses that fail silently: `idioms.md`.

## 8. No equivalent / needs a human

Don't invent replacements for these. Leave a `// MIGRATION:` comment and list the item under "Unmapped" with the
reason. If you use one of the listed alternatives, say so in the report.

| Ext JS | Status | Closest alternative | Tag |
|---|---|---|---|
| Spreadsheet selection model (`gantt_spreadsheet`, `spreadsheet` selModel) | Not available | Cell selection (`selectionMode : { cell : true }`) | G-G |
| `gantt_selectionreplicator` | Not available | `fillHandle` (differs) | G-G |
| `taskBodyTemplate`, `projectLinesConfig.linesFor`, `allowDeselect`, `TaskType` nodes | Removed | see §4 | G-G |
| `DynamicAssignment` scheduling mode | Removed | `FixedDuration` + `effortDriven : true` | G-G |
| Per-calendar `hoursPerDay`/`daysPerWeek`/`daysPerMonth` | Moved to project | `DurationConverterMixin` (see `products/gantt.md`) | G-G |
| `Gnt.widget.calendar.CalendarWindow` / `CalendarPanel` ("Edit working time") | Not available (`CalendarPicker` only selects a calendar; `CalendarEditor` exists in the Scheduler package and ran in a test) | Drop the button and list it, or use `CalendarEditor` | TEST |
| Reminders (Extensible `Reminder`) | Not available | Keep as a custom string field; show a bell with `renderData.iconCls` | TEST |
| `Ext.app.Application`, controllers, Sencha Cmd, `Ext.Loader` | No concept | ES modules + Vite or the framework's build | G-G |

Outside Bryntum's scope — flag for a human; may need the host framework, a third-party library or custom code:

- Ext data proxies beyond plain AJAX/JSON/REST (`direct`, `jsonp`, custom writers, `LocalStorage` proxy). **Ext Direct**
  (`directFn`, `paramOrder`, `Ext.direct.*`): keep the server contract. `CrudManager` and `AjaxStore` don't speak it, so
  write a thin client (`fetch` a POST `{ action, method, data, type : 'rpc', tid }`, read `.result`) and load by assigning
  `project` `resources`/`events` (or `store.data`). Persist from Bryntum handlers: `beforeEventDropFinalize` with
  `context.async = true` + `context.finalize(ok)` for server-validated moves, or an `AjaxStore` subclass whose
  `commit()` maps `store.changes` to the create/update/destroy calls and writes the server id back. Add the mock
  router as a Vite plugin (`configurePreviewServer` too, for `vite preview`) when no backend exists (TEST)
- `Ext.app.ViewModel` `bind : '{...}'` and `formulas` — rewrite as explicit handlers or framework state, item by item
- Ext Charts (`Ext.chart.*`), Pivot grid, D3 adapters — Bryntum's `Chart`/`charts` are not drop-in (UNV). Bryntum Chart
  isn't in the product packages: it ships as the separate package `@bryntum/chart-enterprise` (Enterprise only; 7.x: `@bryntum/chart-thin`); 8.0's `Chart` widget uses AG Charts instead of Chart.js, so check
  its install docs before planning a chart replacement, and otherwise report the chart as unmapped
- `Ext.form.Panel` validation (`vtype`, custom validators, `formBind`) — Bryntum fields have `required`,
  `validateOnInput`, etc., but the model differs (UNV)
- `Ext.util.History` / routing — use the target framework's router
- `Ext.override` / `Ext.define({ override })` of Sch/Gnt internals, and private methods (leading `_` or undocumented)
- Ext state providers (`stateful`, `stateId`) — Bryntum has a `state` mechanism but it isn't equivalent (UNV)
- Ext theming packages, Sass variables and custom UIs — rebuild with CSS variables (`styling.md`)

## 9. Outdated advice (official guides, old blog posts)

Checked against the 7.3.7 and 8.0.0 sources. Correct these whenever you read older material:

- Every `bryntum.<product>.X` UMD global becomes an import from the package that owns `X`: find it in `node_modules/@bryntum/*/entry.js`.
  Examples: `ResourceGrid`, `UndoRedo` from `scheduler-<tier>`; `ButtonGroup`, `CSSHelper`, `DataGenerator` from `core-<tier>`. Plain-script
  `class X extends bryntum.gantt.Column` becomes an ES module `import { Column } from '@bryntum/grid-<tier>'`, then `class X extends Column`.
- Selector strings in a type B app's configs go stale (`rowResize.cellSelector` `.b-sequence-cell` → `.b-row-number-cell`): grep them all against the installed CSS.
- Scheduler `eventRenderer` argument `tplData` → **`renderData`**.
- `eventEdit.editorConfig.showResourceField`, `startTimeConfig`, `endTimeConfig` → don't exist. Use
  `items : { startTimeField : {...}, resourceField : {...} }`, or `null` to remove.
- `extraItems` with `index` → keyed `items` with `weight` (built-ins at 100, 200, ...).
- Gantt assignment fields `taskId`/`resourceId` → **`event`/`resource`**.
- `gantt_projecteditor` "obsolete" → `projectEdit` exists. `gantt_clipboard` "not supported" → `taskCopyPaste` +
  `cellCopyPaste`.
- `.b-rownumber-cell` → **`.b-row-number-cell`**; 7.x renamed selectors to kebab-case throughout.
- `material2-light/dark.css` → **`material3-light/dark.css`**; `high-contrast-light/dark.css` also exists.
- Pre-7.0 combined theme files (`gantt.stockholm.css`) → `gantt.css` + `stockholm-light.css` + FontAwesome CSS
  (separate since 7.0; on 8.0 from `@bryntum/<product>-<tier>` and `@bryntum/core-<tier>/fontawesome`).
- `pan : true` at the widget top level → inside `features`.
- `@bryntum/gantt ^5.x` pins → `@bryntum/gantt-<tier>` at the current 8.x, exact version (7.x: `@bryntum/gantt`).
- Guides that install `@bryntum/<product>` from `npm.bryntum.com`, or a `-trial` alias → public npm tiered package, no login.
- The official Gantt-for-Ext guide's imports (`LocaleHelper`, `Toast`, `Timeline` from `@bryntum/gantt-<tier>`) fail on
  8.0, where a package exports its own classes only. Owners (checked in the 8.0.0 `entry.js` files): `LocaleHelper`,
  `Toast`, `MessageDialog`, `TabPanel`, `PagingToolbar`, `Popup`, `Toolbar`, `Store`, `DateHelper` → `@bryntum/core-<tier>`;
  `Grid`, `TreeGrid`, `Column` → `@bryntum/grid-<tier>`; `Scheduler`, `Timeline`, `PresetManager`, `EventModel` →
  `@bryntum/scheduler-<tier>`; `Gantt`, `ProjectModel`, `TaskModel` (Gantt's own) → `@bryntum/gantt-<tier>`. Always confirm
  in the installed `entry.js` or `*.d.ts`.
- Guides that name Scheduler Pro (`Sch.panel.SchedulerPro`, `@bryntum/schedulerpro`) → Scheduler at the Enterprise tier.
- Button `cls : 'b-raised'` → `rendition : 'filled'`.
- Set an editor title in `beforeEventEditShow` (works on 8.0.0; earlier hooks overwrite it on 7.3.7).
- `tasksData`/`eventsData`/`resourcesData` → `tasks`/`events`/`resources` (the old names log a deprecation warning in 8.0.1).
- `idField : 'Id'` as a store config (Scheduler guide) → silently ignored. Use `static idField = 'Id'` on the model,
  or a `{ name : 'id', dataSource : 'Id' }` field.
