# API mapping: Ext JS → Bryntum 7

Verified against Bryntum **7.3.7**. For a newer installed version, confirm anything tagged other than SRC with the MCP
`search_bryntum_docs` tool (pass the installed `version`) or the docs.

Tags:

- **G-S** / **G-G** — stated in the official Ext → Bryntum Scheduler / Gantt migration guide (corrected where wrong)
- **SRC** — confirmed in the 7.3.7 source (check it in `node_modules/@bryntum/<product>/` for the installed version)
- **DOC** — added for general Ext app migrations; the name is confirmed in the 7.3.7 typings (`*.d.ts`), but the
  mapping hasn't been verified end to end — check config details in the docs before relying on them
- **UNV** — the Bryntum name exists, but equivalence to the Ext behavior is not verified

Product-specific rows only apply to that product. Don't generalize a Gantt row to Scheduler or vice versa.

## 1. Classes and application structure

| Ext JS | Bryntum 7 | Notes | Tag |
|---|---|---|---|
| `Sch.panel.SchedulerGrid` / `Sch.panel.SchedulerPanel` | `Scheduler` | `new Scheduler({ appendTo, ... })` | G-S |
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
| `ftype : 'summary'` | `summary` feature + column `sum` | DOC |
| `ftype : 'groupingsummary'` | `groupSummary` | DOC |
| `rowexpander` / `ftype : 'rowbody'` | `rowExpander` | DOC |
| `bufferedrenderer` | remove — virtual rendering is built in | DOC |
| `clipboard` / `gantt_clipboard` | `cellCopyPaste` (+ `taskCopyPaste : { useNativeClipboard : true }` in Gantt) | G-G, SRC |
| `gridviewdragdrop` (reorder rows) | `rowReorder` (default on in Gantt) | G-G |
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
| `renderer(value, meta, record)` | `renderer({ value, record, cellElement, row, ... })` — return the content | SRC |
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
| grid `select` / `selectionchange` events | `selectionChange` | SRC |
| `itemclick` / `itemdblclick` / `cellclick` | `cellClick` / `cellDblClick` (Grid); `eventClick` / `eventDblClick` (Scheduler); `taskClick` (Gantt) | DOC |
| `record.get('Name')` / `record.set('Name', v)` | `record.name` / `record.name = v` (`get()`/`set()` also exist) | SRC |
| `grid.getStore()` / `getSelectionModel().getSelection()` | `grid.store` / `grid.selectedRecords` | DOC |
| `store.getAt(i)`, `store.getById(id)`, `store.add`, `store.remove` | same names | DOC |
| `store.each(fn)` | `store.forEach(fn)` | DOC |
| `listeners : { x : fn, scope }` | `listeners : { x : fn, thisObj }` or `widget.on({ x })` (returns a detacher). A subclass method named `on<Event>` is already a listener (`callOnFunctions`), so don't also register it | G-G, SRC |
| button `handler` | `onClick` / `onAction`; string `'up.methodName'` calls the method on the nearest ancestor that has it, passing the click event. See `patterns.md` §3 | G-G, SRC |
| `Ext.getCmp(id)` / `lookupReference(ref)` / `down('#id')` | `widgetMap.ref`, `Widget.getById(id)` | G-G, SRC |
| `Ext.XTemplate` | function returning a template literal; escape data with `StringHelper.xss` tagged template | SRC |
| `Ext.Date.format(d, 'Y-m-d H:i')` | `DateHelper.format(d, 'YYYY-MM-DD HH:mm')` — **tokens differ** (PHP-style → moment-style: `Y`→`YYYY`, `m`→`MM`, `d`→`DD`, `H`→`HH`, `i`→`mm`, `s`→`ss`, `g:i A`→`h:mm A`, `D`→`ddd`, `l`→`dddd`, `M`→`MMM`, `F`→`MMMM`, `j`→`D`, `n`→`M`) | SRC |
| `Ext.Date.add(d, Ext.Date.DAY, 1)` | `DateHelper.add(d, 1, 'day')` | SRC |
| `Ext.Date.clearTime(d)` | `DateHelper.clearTime(d)` | SRC |
| `suspendLayouts` / `resumeLayouts` | `suspendRefresh()` / `resumeRefresh()` | UNV |
| `expandAll`, `collapseAll`, `zoomIn`, `zoomOut`, `zoomToFit`, `shiftPrevious`, `shiftNext` | same names. Wrap them in a handler rather than using `'up.shiftNext'` (the event object would become `amount`) | G-G, SRC |

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
- Ext Charts (`Ext.chart.*`), Pivot grid, D3 adapters — Bryntum's `Chart`/`charts` are not drop-in (UNV)
- `Ext.form.Panel` validation (`vtype`, custom validators, `formBind`) — Bryntum fields have `required`,
  `validateOnInput`, etc., but the model differs (UNV)
- `Ext.util.History` / routing — use the target framework's router
- `Ext.override` / `Ext.define({ override })` of Sch/Gnt internals, and private methods (leading `_` or undocumented)
- Ext state providers (`stateful`, `stateId`) — Bryntum has a `state` mechanism but it isn't equivalent (UNV)
- Ext theming packages, Sass variables and custom UIs — rebuild with CSS variables (`styling.md`)

## 9. Outdated advice (official guides, old blog posts)

Verified against the 7.3.7 source. Correct these whenever you read older material:

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
  (separate since 7.0).
- `pan : true` at the widget top level → inside `features`.
- `@bryntum/gantt ^5.x` pins → current 7.x, exact version.
- Button `cls : 'b-raised'` → `rendition : 'filled'`.
- Setting an editor title in `eventEditBeforeSetRecord` is overwritten in 7.3.7; set it in `beforeEventEditShow`.
- `tasksData`/`eventsData`/`resourcesData` → `tasks`/`events`/`resources`.
- `idField : 'Id'` as a store config (Scheduler guide) → silently ignored. Use `static idField = 'Id'` on the model,
  or a `{ name : 'id', dataSource : 'Id' }` field.
