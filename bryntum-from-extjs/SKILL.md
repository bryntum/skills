---
name: bryntum-from-extjs
description: >
  Bryntum's API shares vocabulary with Ext JS (Store, Model, Container, items, tbar, type
  aliases, listeners, cls, flex, renderer) but is a different library. Use alongside the
  `bryntum` skill whenever Ext JS habits could leak in — `store.first()`, `component.down()`,
  `store.get(id)`, `Ext.getCmp`, `Ext.create`, `Ext.define`, `xtype`, `dataIndex`, `renderTo`,
  `Ext.Msg`, `Ext.Ajax`, `grid.getStore()`, `getSelectionModel()`, `store.sync()`, or when the
  user mentions Ext JS, ExtJS, Sencha, `Sch.panel.SchedulerGrid` or `Gnt.panel.Gantt`.
metadata:
  tags: bryntum, extjs, sencha, migration, store, widget, api-differences
---

# Bryntum for Ext JS developers

Bryntum used to ship Ext Scheduler (`Sch.panel.SchedulerGrid`) and Ext Gantt
(`Gnt.panel.Gantt`) as Ext JS add-ons. Those products are discontinued. Today's Scheduler,
Scheduler Pro, Gantt, Grid, Calendar and TaskBoard are standalone ES modules with no Ext JS
dependency and no shared code, so answers and forum threads under `Sch.*` / `Gnt.*` don't apply.

`Ext.create` rarely leaks in by accident. The real risk is the shared vocabulary: both
libraries have a `Store` with `first`, a `Container` with `items`, a column `renderer`, and a
`type` that looks like `xtype`. Several near-misses fail silently or with a confusing error
rather than "Ext is not defined".

Verified against Bryntum 7.3.6 source. The full Ext → Bryntum lookup (construction, component
queries, store, model, tree nodes, config names, dialogs, helpers, selection) is in
[references/api-map.md](references/api-map.md).

## The near-misses that bite hardest

| Ext JS | Bryntum | Why it hurts |
|---|---|---|
| `store.first()` | `store.first` | A getter; calling it throws `store.first is not a function`. `last` is a getter too |
| `container.down('#save')` | `container.widgetMap.save` (with `ref : 'save'`) | There is no `down()`, though `up()` exists |
| `store.find('name', 'Bob')` | `store.findRecord('name', 'Bob')` | `find(fn, searchAllRecords)` takes a predicate; a field name returns `undefined` with no error. `query` is a predicate too |
| `store.get(5)` | `store.getById(5)` | No `get()` |
| `store.each(fn)` | `store.forEach(fn)` | No `each()` |
| `store.clearFilter()` | `store.clearFilters()` | Plural |
| `Toast.show({ type : 'success' })` | `Toast.show('Saved')` | `type` is the registered widget type; setting it throws `Can not mutate 'type' config to different value` |
| column `dataIndex` | column `field` | `dataIndex` doesn't exist anywhere in Bryntum |
| `listeners : { …, scope : this }` | `listeners : { …, thisObj : this }` | Same idea, different key |
| button `handler` | `onClick` | Any `onXxx` config becomes a listener for `xxx`. `onClick : 'up.onSave'` calls `onSave` on an owning widget, like a ViewController method name |
| positional `renderer(value, metaData, record, …)` | `renderer : ({ record, value, column, row, cellElement }) => …` | One object argument |
| `node.insertChild(index, child)` | `node.insertChild(child, before)` | Arguments reversed, and the second is the sibling record to insert before, not an index |
| `grid.getView().refresh()` | `grid.refreshRows()` | Grid has no `refresh()`; Calendar, TaskBoard, Scheduler and Gantt do |

Also watch:

- `Widget.query(sel)` returns the first match; `Widget.queryAll(sel)` returns an array. The
  instance `container.query` / `queryAll` accept a function only.
- Assigning a property that isn't in `static fields` (`record.foo = 1`) creates a plain own
  property: no dirty flag, no sync, no persistence. Declare the field.
- In cell-selection mode, `grid.isSelected(record)` doesn't report rows selected via their
  cells; use `grid.selectedRecords.includes(record)`.
- In a grouped store, `find(fn)` skips group headers and does find records inside collapsed
  groups. Those records aren't in `store.records` and have no row until the group is expanded
  (`grid.features.group.toggleCollapse(record, false)`). Pass `true` as the second argument to
  include rows hidden by a filter.
- Assigning `store.data = arr` replaces records but keeps sorters, filters, groupers and collapsed
  groups, unlike a fresh Ext `loadData`. Call `clearSorters()`, `clearFilters()` and
  `grid.expandAll()` to get back to the original view.
- `DateHelper.format` uses Unicode-style tokens (`'YYYY-MM-DD'`), not Ext's PHP-style ones.
- `TabPanel.activeTab` accepts a widget but returns a number index; use `activeItem` for the widget.

## Things with no Ext JS counterpart

- The `ProjectModel` and scheduling engine. In Scheduler Pro and Gantt a constraint solver
  recalculates dates from dependencies, calendars and constraints. Don't write `endDate` to move
  a task; set `startDate` and `duration`, or a constraint, and let the engine resolve.
- `features : { … }` is much broader than Ext `plugins`; see `bryntum-features`.

Don't guess a Bryntum name from its Ext JS equivalent — check `references/api-map.md`, then the
docs or MCP search described in the `bryntum` skill.

## Related

- `bryntum` — core skill; install first.
- `bryntum-features` — the built-in feature for what the user is describing.
- `bryntum-crud` — CrudManager and AjaxStore backends (the `proxy` replacement).
