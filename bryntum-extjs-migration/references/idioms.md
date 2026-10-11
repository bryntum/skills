# Ext idioms and silent near-misses

Method- and property-level Ext → Bryntum differences, for code being ported line by line. Verified against the
Bryntum 7.3.7 source. Rows where the name and behaviour are identical are omitted. Class, config, plugin and column
mappings are in `api-mapping.md`.

## The near-misses that fail silently

Both libraries have a `Store` with `first`, a `Container` with `items`, a column `renderer` and a `type` that looks
like `xtype`. These look right and fail quietly or with a confusing error:

| Ext JS | Bryntum | Why it hurts |
|---|---|---|
| `store.first()` / `last()` | `store.first` / `store.last` | Getters: calling them throws `store.first is not a function` |
| `store.find('name', 'Bob')` | `store.findRecord('name', 'Bob')` | `find(fn, searchAllRecords)` takes a predicate: a field name returns `undefined`, no error. `query` is a predicate too |
| `store.get(5)` | `store.getById(5)` | No `get()` |
| `store.each(fn)` | `store.forEach(fn)` | No `each()` |
| `store.clearFilter()` | `store.clearFilters()` | Plural |
| `store.sort('name', 'DESC')` | `store.sort('name', false)` | 2nd argument is `ascending`; 3rd `add : true` multi-sorts |
| `container.down('#save')` | `container.widgetMap.save` (with `ref : 'save'`) | No `down()`, though `up()` exists |
| `node.insertChild(index, child)` | `node.insertChild(child, before)` | Arguments reversed; `before` is a sibling record, not an index |
| `grid.getView().refresh()` | `grid.refreshRows()` | Grid has no `refresh()` (Scheduler, Gantt, Calendar, TaskBoard do) |
| `Toast.show({ type : 'success' })` | `Toast.show('Saved')` | `type` is the widget type: setting it throws `Can not mutate 'type' config` |
| column `dataIndex` | column `field` | `dataIndex` exists nowhere in Bryntum |
| `listeners : { …, scope : this }` | `listeners : { …, thisObj : this }` | Same idea, different key |
| positional `renderer(value, metaData, record, …)` | `renderer : ({ value, record, column, row, cellElement }) => …` | One object argument |

Also watch:

- `record.foo = 1` for a field not in `static fields` creates a plain property: no dirty flag, no sync, no save.
  Declare the field.
- `store.data = arr` (Ext `loadData`) keeps sorters, filters, groupers and collapsed groups. Call `clearSorters()`,
  `clearFilters()` and `grid.expandAll()` to get back to the original view.
- TextField `change` fires on blur by default (`keyStrokeChangeDelay : 0`), not per keystroke. Search-as-you-type →
  `onInput`.
- `TabPanel.activeTab` accepts a widget but returns a number; use `activeItem` for the widget.
- In cell-selection mode, `grid.isSelected(record)` misses rows selected through their cells; use
  `grid.selectedRecords.includes(record)`.
- In a grouped store, `find(fn)` skips group headers and finds records inside collapsed groups, which aren't in
  `store.records` and have no row until expanded (`grid.features.group.toggleCollapse(record, false)`).
- `Widget.query(sel)` returns the first match; `Widget.queryAll(sel)` an array. Instance `container.query` /
  `queryAll` take a function only.

## Construction and classes

| Ext JS | Bryntum | Note |
|---|---|---|
| `requires` / `Ext.require` | `import { Grid } from '@bryntum/grid'` | |
| `renderTo : 'app'` | `appendTo : 'app'` | Also `insertBefore`, `insertFirst`, `adopt`. No `renderTo` |
| `config` + generated `getFoo()` / `setFoo()` | `static configurable` + property access (`widget.foo = 1`) | No generated getters/setters |
| `applyFoo` / `updateFoo` | `changeFoo(value, old)` / `updateFoo(value, old)` | `change` normalizes, `update` reacts |
| `beforerender` / `afterrender` | `paint` event | Fires with `{ firstPaint }` |

## Finding components

| Ext JS | Bryntum | Note |
|---|---|---|
| `Ext.getCmp('id')` | `Widget.getById('id')` | |
| `Ext.ComponentQuery.query('grid')` | `Widget.queryAll('grid')` | Also `bryntum.queryAll` from the console |
| `cmp.up('panel')` | `widget.up('panel')` | Type string or function, not CSS-ish. `closest()` includes the widget itself |
| `cmp.down('button')` | `container.query(w => w.isButton)` | Function only |
| `cmp.getComponent('id')` | `container.getWidgetById('id')` | |
| `cmp.items.getAt(0)` | `container.items[0]` | `items` is a plain array once built |

Prefer `ref` over `id`: `id` must be unique across the document, `ref` need not be.

## Store

| Ext JS | Bryntum | Note |
|---|---|---|
| `grid.getStore()` | `grid.store` | |
| `store.getCount()` | `store.count` | `getCount(options)` also exists |
| `store.getRange()` | `store.getRange(start, end)` or `store.records` | |
| `store.find('name', x)` | `store.findRecord('name', x)` or `findByField('name', x)` | `find(fn, true)` includes filtered-out rows |
| `store.query('name', /bob/)` | `store.query(fn)` | |
| `store.filter('city', 'Paris')` | `store.filter({ property : 'city', value : 'Paris', operator : '=' })` | Also a predicate, an array, or `{ filters, replace }`. One filter off: `removeFilter(idOrInstance)` |
| `store.group('city')` | same | Also `addGrouper`, `removeGrouper`, `clearGroupers` |
| `store.sync()` | `store.commit()` (AjaxStore) | `acceptChanges()` marks clean without a request; `revertChanges()` rolls back |
| `store.getModifiedRecords()` | `store.changes` (`{ added, modified, removed }`) / `store.hasChanges` | |

## Record and tree node

| Ext JS | Bryntum | Note |
|---|---|---|
| `record.get('name')` / `set(…)` | same, or `record.name` / `record.name = v`; `getValue(f)` / `setValue(f, v)` | Declared fields only |
| `record.dirty` | `record.isModified` | Per field: `isFieldModified(name)` |
| `record.getChanges()` | `record.modifications` | |
| `record.reject()` / `commit()` | `record.revertChanges()` / `clearChanges()` | |
| `record.copy()` | `record.copy(newIdOrData, deep)` | |
| `store.remove(record)` | same, or `record.remove()` | |
| `node.isLeaf()` | `node.isLeaf` | Getter |
| `node.getDepth()` | `node.childLevel` | |
| `node.cascadeBy(fn)` | `node.traverse(fn, skipSelf)` | |
| `node.expand()` | `await grid.features.tree.expand(record)` | Also `collapse`, `toggleCollapse`, `expandTo`, `expandToLevel` |
| grouping `expand(name)` / `collapseAll()` | `await grid.features.group.toggleCollapse(record, false)` / `grid.collapseAll()` | Any record in the group, or its header |

## Dialogs, helpers, masking

| Ext JS | Bryntum | Note |
|---|---|---|
| `Ext.toast('Saved')` | `Toast.show('Saved')` | Config form: `html`, `timeout`, `showProgress` (not `type`) |
| `Ext.Ajax.request({ url, success })` | `await AjaxHelper.get(url, { parseJson : true })` | Also `post`, `fetch`; read `parsedJson` |
| `Ext.util.Format.currency` | NumberColumn `format : { style : 'currency', currency : 'USD' }` | |
| `Ext.util.Format.htmlEncode` | `StringHelper.encodeHtml` | Also `stripHtmlTags`, the `xss` tagged template |
| `Ext.Array` / `Ext.Object` | `ArrayHelper` / `ObjectHelper` | |
| `cmp.mask('Loading')` | `widget.masked = 'Loading'` | Or `Mask.mask(text, element)` |
| `Ext.form.Panel` values | `container.values` | Validity via `isValid` |

## Refresh and selection

| Ext JS | Bryntum | Note |
|---|---|---|
| refresh one column | `grid.refreshColumn(column)` | |
| `getSelectionModel().getSelection()` | `grid.selectedRecords` | Also `selectedRecord`, `selectedCells` |
| `sm.select(rec)` / `deselect(rec)` / `deselectAll()` | `grid.selectRow({ record })` / `deselectRow(record)` / `deselectAll()` | |
| default single-row selection (`rowmodel`, `SINGLE`) | `selectionMode : { multiSelect : false }` | Bryntum defaults to `multiSelect : true` |

## No Ext counterpart

- Scheduler Pro and Gantt run a scheduling engine (`ProjectModel`): it recalculates dates from dependencies,
  calendars and constraints. Don't write `endDate` to move a task; set `startDate` and `duration` (or a constraint)
  and let the engine resolve, then `await project.commitAsync()` before reading results.
