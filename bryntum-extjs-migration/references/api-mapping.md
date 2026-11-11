# API mapping: Ext JS → Bryntum 8

Source names checked against the Bryntum **8.0.0** packages (7.3.7 for rows that say so; 7.x differences are in
`v7.md`). For a newer installed version, confirm anything tagged other than SRC with the MCP
`search_bryntum_docs` tool (pass the installed `version`) or the docs.

Tags:

- **G-S** / **G-G** — stated in the official Ext → Bryntum Scheduler / Gantt migration guide (corrected where wrong)
- **SRC** — confirmed in the 8.0 / 7.3.7 source (check it in `node_modules/@bryntum/<product>-<tier>/` for the installed version)
- **DOC** — added for general Ext app migrations; the name is confirmed in the 7.3.7/8.0 typings (`*.d.ts`), but the
  mapping hasn't been verified end to end — check config details in the docs before relying on them
- **UNV** — the Bryntum name exists, but equivalence to the Ext behavior is not verified

Product-specific rows only apply to that product. Don't generalize a Gantt row to Scheduler or vice versa.

## 0. Tiers: what each Ext feature needs on 8.0

The tiers are the **Community < Core < Enterprise** tiers. The Core tier was called Business in 8.0.0: those packages and the older
docs say `-business` / `'business'`; 8.0.1 renamed them to `-core` / `'core'`. A `-business` app is already on the Core tier, and
`-business` and `-core` must not be mixed. Run `npm view @bryntum/<product>-core version` and fall back to `-business` if it 404s
(8.0.1 not out yet), and read "core" as "business" for those packages. `@bryntum/core-core` is the Core package at the Core tier,
not a typo.

A feature above the installed tier is skipped, and a dialog names the tier it needs. Check each ported plugin before
you promise it. **Source of truth: what the package ships.** A feature belongs to the lowest tier whose published package
contains its class file (checked against the 8.0.0 `-community`, `-business` and `-enterprise` tarballs; this also matches the
`feature-availability.md` guides, 164 of 164 features). Do not use `static tier` in the source: for ~25 features it says
Business (`'core'` in the suite) although only the Enterprise package contains them. Anything not named below is in Community.

| Product | Core tier and up | Enterprise tier only |
|---|---|---|
| Grid | `filter` (header-menu filter), `group`, `cellCopyPaste`, `rowCopyPaste`, `cellTooltip`, `columnDragToolbar`, `columnRename`, `pinColumns`, `stickyCells`, `treeGroup` | `summary`, `groupSummary`, `rowExpander`, `rowReorder`, `lockRows`, `split`, `mergeCells`, `charts`, `pdfExport`, `print`, `rowEdit`, `fillHandle`, `rowResize`, AI |
| Scheduler | `dependencyEdit`, `dependencyMenu`, `eventCopyPaste`, `eventDragSelect`, `eventFilter`, `headerZoom`, `labels`, `nonWorkingTime`, `pan`, `percentBar`, `resourceColumnReorder`, `resourceTimeRanges`, `scheduleContext`, `scrollButtons`, `simpleEventEdit`, `stickyEvents`, `timeRanges`, `timeSelection` | `taskEdit`, `eventClustering`, `resourceTimeRangeClustering`, `eventNonWorkingTime`, `resourceNonWorkingTime`, `summary`, `timelineSummary`, `treeSummary`, `groupSummary`, `tickCells`, `lockRows`, `rowReorder`, `split`, `pdfExport`, `print`, `eventBuffer`, `eventSegments` (+ drag/resize), `nestedEvents`, `groupRollups`, `resourceEdit`, `calendarHighlight`, `timeSpanHighlight`, `allocationCellEdit`, `allocationCopyPaste`, `rowResize`, `versions`, AI |
| Gantt | `taskEdit`, `taskCopyPaste`, `taskNonWorkingTime`, `progressLine`, `projectLines`, `indicators`, `labels`, `parentArea`, `scrollButtons`, `treeGroup` | `summary`, `pdfExport`, `print`, `baselines`, `criticalPaths`, `projectEdit`, `rollups`, `timelineChart`, `taskSegmentDrag`/`taskSegmentResize`, MS Project export, `versions`, AI |
| Calendar | recurring events (data layer), `eventCopyPaste`, `externalEventSource`, `timeRanges`, `weekExpander` | `eventBuffer`, `loadOnDemand`, `print`, AI; views `dayagenda`, `monthagenda`, `resource`, `dayresource`, `monthgrid`, `yearplanner`, `multiweek` |
| TaskBoard | swimlanes, `columnFilter`, `columnRename`, `columnResize`, `filterBar`, `simpleTaskEdit`; card items `jsx`, `resourceAvatars`, `tags`, `todoList` | `columnDrag`, `columnLock`, `columnSort`, `swimlaneDrag`, card items `chart`, `collapse`, AI |

Open question for the Bryntum devs (not yet answered): `gantt-business` also contains Grid's `rowExpander` and `rowReorder`, which the
Grid docs call Enterprise and the Grid Business package does not ship. Don't promise them at the Core tier until confirmed. These
tier results are for 8.0.0; re-check against the `-core` packages once 8.0.1 is published.

Engine and data layer:

- **Gantt Community** runs a lighter engine (base task: dates, name, `percentDone`, WBS): no working time, constraints,
  critical path, lag/lead, effort or scheduling direction. **Gantt Core** has the full engine (constraints,
  dependencies, calendars). **Gantt Enterprise** adds baselines and project revisions.
- **Calendar Community** has no recurring events (the Core tier adds them, with lazy loading): an Ext Calendar with recurrence needs the Core tier.
- **Scheduler Core** adds recurring events and lazy loading to the plain data model. **Scheduler Enterprise** swaps in
  the scheduling engine (constraints, effort, `percentDone`, calculated dates) and the default editor becomes `taskEdit`
  (`taskEdit` ships only in the Enterprise package; the Community and Core tiers use `eventEdit`). It is the former Scheduler Pro.
- Models and stores resolve to the tier's class from the plain name (`TaskModel` → `TaskModelEnterprise`). To run the
  Enterprise package at a lower tier set `tier : 'core'` on the widget (`'business'` on the 8.0.0 packages). Don't `instanceof` across tiers: use the `is<Name>` flags.
- A field a lower tier doesn't have reads as `undefined`, silently. Re-check Ext data fields such as `Effort`,
  `ConstraintType` and `Baselines` against the tier.
- Re-check a feature in the installed package: it is there if `node_modules/@bryntum/<product>-<tier>/lib/<Product>/feature/<Name>.js` exists (also look under `lib/Grid` and `lib/Scheduler` for Gantt), not by its `static tier`.

## 1. Classes and application structure

| Ext JS | Bryntum 8 | Notes | Tag |
|---|---|---|---|
| `Sch.panel.SchedulerGrid` / `Sch.panel.SchedulerPanel` | `Scheduler` | `new Scheduler({ appendTo, ... })`. With dependencies, assignments or calendars: Enterprise tier (7.x: `SchedulerPro`, see `v7.md`) | G-S |
| `Gnt.panel.Gantt` | `Gantt` | | G-G |
| `Ext.grid.Panel` / `Ext.grid.Grid` (Modern) | `Grid` | | DOC |
| `Ext.tree.Panel` | `TreeGrid` (or `Grid` with a tree store + `type : 'tree'` column) | | DOC |
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
| `vtype`, custom `validator` | none direct — see §8 | UNV |

## 3. Scheduler-specific configs

| Ext Scheduler | Bryntum Scheduler | Tag |
|---|---|---|
| `title` | keep — Scheduler is a Panel in 7.x (`title`, `tools`, `tbar` work). The guide's "remove" is outdated | SRC |
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
| `allowOverlap`, `eventBarTextField`, `readOnly`, `zoomOnMouseWheel` | same names | UNV |
| `mode : 'vertical'` / `orientation` | `mode : 'vertical'` | UNV |
| `dndValidatorFn` | `features.eventDrag.validatorFn` | UNV |
| `beforeeventdrop` listener returning `false` | `beforeEventDropFinalize` (there is no `beforeEventDrop` in 7.x): set `context.valid = false`, or `context.async = true` + `context.finalize(bool)` | SRC |
| `resizeValidatorFn` | `features.eventResize.validatorFn` | UNV |
| `createValidatorFn` | `features.eventDragCreate.validatorFn` | UNV |
| `multiSelect` (events) | `multiEventSelect` | UNV |
| `resourceImagePath` | `resourceImagePath` or `resourceImages : { path, extension }` | SRC |
| event colors (renderer-injected) | `eventColor` field/config + `eventStyle` (`'tonal'`, `'filled'`, `'bordered'`, `'traced'`, `'outlined'`, `'indented'`, `'line'`, `'dashed'`, `'minimal'`, `'rounded'`) | SRC |

## 4. Gantt-specific configs

| Ext Gantt | Bryntum Gantt | Tag |
|---|---|---|
| `showTodayLine` | `features.timeRanges.showCurrentTimeLine : true` | G-G |
| `loadMask` | `loadMask` | G-G |
| `enableProgressBarResize` | `percentBar` feature | G-G, UNV for Gantt |
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

## 5. Plugins / grid features → features

| Ext | Bryntum feature | Tag |
|---|---|---|
| `cellediting` / `scheduler_treecellediting` (`clicksToEdit`) | `cellEdit` (default on in Grid-based widgets) | G-G, SRC |
| `rowediting` | `rowEdit` (7.3.7 has it; compare its UX with the Ext row editor) | DOC |
| `gridfilters` | `filter` (header menu) or `filterBar` (filter row) | G-G, SRC |
| `ftype : 'grouping'` | `group` feature (`features : { group : 'field' }`). `hideGroupedHeader` has no Group config: set `hidden : true` on the grouped column (`hideGroupedColumns` exists only on `treeGroup`) | SRC |
| `ftype : 'summary'` | `summary` feature + column `sum` (Enterprise tier, §0) | DOC |
| `ftype : 'groupingsummary'` | `groupSummary` (Enterprise tier) | DOC |
| `rowexpander` / `ftype : 'rowbody'` | `rowExpander` (Grid: Enterprise tier, §0) | DOC |
| `bufferedrenderer` | remove — virtual rendering is built in | DOC |
| `clipboard` / `gantt_clipboard` | `cellCopyPaste` (+ `taskCopyPaste : { useNativeClipboard : true }` in Gantt) | G-G, SRC |
| `gridviewdragdrop` (reorder rows) | `rowReorder` (Enterprise tier in Grid and Scheduler; default on in Gantt) | G-G |
| drag between grids / onto a timeline | `DragHelper` — see the `bryntum-drag-and-drop` skill | SRC |
| context menus (`itemcontextmenu` listener, `advanced_taskcontextmenu`, `scheduler_eventcontextmenu`) | `cellMenu`, `headerMenu` (Grid); `eventMenu` (Scheduler); `taskMenu` (Gantt) — on by default | G-G, SRC |
| `scheduler_pan` | `pan` | G-G, SRC |
| Scheduler event editor plugin | `eventEdit`, customize with keyed `items` + `weight` | SRC |
| `gantt_taskeditor` (`taskFormClass`) | `taskEdit`, customize `items : { generalTab : { items : {...} } }` | G-G |
| `gantt_projecteditor` | `projectEdit` (exists in 7.3; the guide says obsolete) | SRC |
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
| `xtype : 'actioncolumn'` (`items`, `handler`) | `type : 'action'`, `actions : [{ cls, tooltip, onClick }]` | DOC |
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
| `Ext.app.Application`, controllers, Sencha Cmd, `Ext.Loader` | No concept | ES modules + Vite or the framework's build | G-G |

Outside Bryntum's scope — flag for a human; may need the host framework, a third-party library or custom code:

- Ext data proxies beyond plain AJAX/JSON/REST (`direct`, `jsonp`, custom writers, `LocalStorage` proxy)
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
- Setting an editor title in `eventEditBeforeSetRecord` is overwritten in 7.3.7; set it in `beforeEventEditShow`.
- `tasksData`/`eventsData`/`resourcesData` → `tasks`/`events`/`resources`.
- `idField : 'Id'` as a store config (Scheduler guide) → silently ignored. Use `static idField = 'Id'` on the model,
  or a `{ name : 'id', dataSource : 'Id' }` field.
