# Ext JS → Bryntum API map

Rows where the name and behaviour are identical in both libraries are omitted.

## Construction and the class system

| Ext JS | Bryntum | Note |
|---|---|---|
| `Ext.define('My.Grid', { extend : 'Ext.grid.Panel' })` | `class MyGrid extends Grid { static $name = 'MyGrid'; }` | Plain ES classes. `$name` is used for the feature/type registry, not a namespace |
| `Ext.create('Ext.grid.Panel', cfg)` | `new Grid(cfg)` | No string-based factory for top-level construction |
| `xtype : 'button'` | `type : 'button'` | Declare `static type = '…'` and call `MyWidget.initClass()` once; without it, `type` lookup throws `Invalid type name … passed to Widget factory` |
| `requires` / `Ext.require` | `import { Grid } from '@bryntum/grid'` | |
| `renderTo : 'app'` | `appendTo : 'app'` | Also `insertBefore`, `insertFirst`, `adopt`. No `renderTo` |
| `initComponent()` / `apply*` | `static configurable = { … }` + `changeFoo(value, old)` / `updateFoo(value, old)` | `change` normalizes, `update` reacts |
| `initComponent()` for one-off setup | `construct(config) { super.construct(config); … }` | Call `super.construct` first |
| `config` + generated `getFoo()` / `setFoo()` | `static configurable` + property access (`widget.foo = 1`) | No generated getter/setter methods |
| `Ext.data.Model` `fields : [...]` | `class MyModel extends Model { static fields = ['name', { name : 'due', type : 'date' }]; }` | |

## Finding components

| Ext JS | Bryntum | Note |
|---|---|---|
| `Ext.getCmp('myGrid')` | `Widget.getById('myGrid')` | |
| `Ext.ComponentQuery.query('grid')` | `Widget.queryAll('grid')` | `Widget.query` returns the first match only |
| `cmp.up('panel')` | `widget.up('panel')` | Selector is a type string or function, not CSS-ish. `closest()` includes the widget itself |
| `cmp.down('button')` | `container.query(w => w.isButton)` | Instance `query` / `queryAll` take a function only |
| `cmp.down('#saveBtn')` | `container.widgetMap.saveBtn` (with `ref : 'saveBtn'`) | |
| `cmp.getComponent('id')` | `container.getWidgetById('id')` | |
| `cmp.child()` / `cmp.items.getAt(0)` | `container.items[0]` | `items` is a plain array once built |

Static selectors accept `'grid'`, `'#myScheduler'`, `'[ref=timeline]'` or a function, and are
aliased globally as `bryntum.query` / `bryntum.queryAll` (handy from the console). Prefer `ref`
over `id`: `id` must be unique across the document, `ref` need not be.

## Store

| Ext JS | Bryntum | Note |
|---|---|---|
| `store.first()` / `last()` | `store.first` / `store.last` | Getters |
| `store.getCount()` | `store.count` | `getCount(options)` also exists |
| `store.get(id)` | `store.getById(id)` | |
| `store.indexOf(rec)` | `store.indexOf(recordOrId)` | Also accepts an id |
| `store.getRange()` | `store.getRange(start, end)` or `store.records` | |
| `store.each(fn)` | `store.forEach(fn)` | `map`, `reduce` too |
| `store.find('name', 'Bob')` | `store.findRecord('name', 'Bob')` or `findByField('name', 'Bob')` | `find(fn, searchAllRecords)` takes a predicate; `true` includes filtered-out rows. When grouped it skips headers and finds records in collapsed groups, which have no row until expanded |
| `store.query('name', /bob/)` | `store.query(fn)` | Predicate |
| `store.filter('city', 'Paris')` | `store.filter({ property : 'city', value : 'Paris', operator : '=' })` | Also a predicate, an array, or `{ filters, replace }` |
| `store.clearFilter()` | `store.clearFilters()` | Single removal: `removeFilter(idOrInstance)` |
| `store.sort('name', 'DESC')` | `store.sort('name', false)` | Second arg is `ascending`; third `add : true` for multi-sort |
| `store.group('city')` | same | Also `addGrouper` / `removeGrouper` / `clearGroupers` |
| `store.insert(index, records)` | `store.insert(index, records)` | |
| `store.sync()` | `store.commit()` (on `AjaxStore`) | `acceptChanges()` marks clean without a request; `revertChanges()` rolls back |
| `store.getModifiedRecords()` | `store.changes` / `store.hasChanges` | `changes` is `{ added, modified, removed }` |
| `proxy : { type : 'ajax', url }` | `new AjaxStore({ readUrl, createUrl, updateUrl, deleteUrl })` | `await store.load()`; `autoLoad`, `autoCommit` |
| `grid.getStore()` | `grid.store` | |
| `store.loadData(arr)` | `store.data = arr` | Keeps sorters, filters, groupers and collapsed groups; clear them yourself to reset. `loadData(data, action)` also exists |

## Model / record

| Ext JS | Bryntum | Note |
|---|---|---|
| `record.get('name')` / `set(…)` | same, or `record.name` / `record.name = v` | Declared fields only |
| — | `record.getValue(f)` / `setValue(f, v)` | Field-name-driven variants |
| `record.dirty` | `record.isModified` | |
| `record.getChanges()` | `record.modifications` | Also `isFieldModified(name)` |
| `record.reject()` | `record.revertChanges()` | |
| `record.commit()` | `record.clearChanges()` | |
| `record.copy()` | `record.copy(newIdOrData, deep)` | |
| `store.remove(record)` | also `record.remove()` | |

### Tree nodes

| Ext JS | Bryntum | Note |
|---|---|---|
| `node.appendChild(child)` | `node.appendChild(child, silent, options)` | |
| `node.insertChild(index, child)` | `node.insertChild(child, before, silent, options)` | `before` is a sibling record, not an index |
| `node.removeChild(child)` | `node.removeChild(childRecords, isMove, silent, options)` | |
| `node.isLeaf()` | `node.isLeaf` | Getter |
| `node.getDepth()` | `node.childLevel` | |
| `node.cascadeBy(fn)` | `node.traverse(fn, skipSelf, options)` | |
| grouping `expand(name)` / `collapseAll()` | `await grid.features.group.toggleCollapse(memberRecord, false)` / `grid.expandAll()` / `grid.collapseAll()` | Pass any record in the group, or its header. `expandAll` / `collapseAll` are added to the grid by the `group` feature |
| `node.expand()` | `await grid.features.tree.expand(record)` | Driven by the `tree` feature. Also `collapse`, `toggleCollapse`, `expandAll`, `collapseAll`, `expandTo`, `expandToLevel` |

## Config names

| Ext JS | Bryntum | Note |
|---|---|---|
| `xtype` | `type` | |
| column `dataIndex` | column `field` | |
| column `locked : true` | same (sugar for `region : 'locked'`) | |
| button `handler` | `onClick` | Any `onXxx` config becomes a listener for `xxx` |
| ViewController method name (`handler : 'onSave'`) | `onClick : 'up.onSave'` | Resolved on an owning widget |
| TextField `change` on each keystroke | `onInput` | By default (`keyStrokeChangeDelay : 0`) `change` fires on blur, not per keystroke; use `input` for search-as-you-type |
| `listeners : { click, scope }` | `listeners : { click, thisObj }` | `buffer` and `throttle` work as in Ext |
| `cmp.on('click', fn, scope)` | `widget.on({ click : fn, thisObj : this })` | `on`, `un`, `once`, `trigger`; `on()` returns a detacher |
| — | `dataset`, `weight`, `tooltip`, `ref`, `masked` | Extra common widget configs |
| toolbar `'->'` / `'|'` | same | Filler / separator in `tbar` / `bbar` |
| `layout : 'hbox'` | `layout : 'box'` (alias `'hbox'`) | Available: `default`, `box`/`hbox`, `vbox`, `card`, `fit` |
| `Ext.tab.Panel` `activeTab` | `TabPanel.activeTab` | Setter takes a widget, getter returns a number; use `activeItem` for the widget |
| `Ext.window.Window` | `Popup` | No `window` type |
| `Ext.form.Panel` | `Panel` / `Container` with field items | No `form` type. Values via `container.values` / `getValues()`, validity via `isValid` |
| ComboBox `store` + `displayField` + `valueField` | `Combo` with `items` (or `store`), `displayField` (default `'text'`), `valueField` | |
| `beforerender` / `afterrender` | `paint` event | Fires with `{ firstPaint }` |

Renderers return a string, a `DomConfig` object, or (with the React wrapper) JSX. Return
`undefined` to skip rendering entirely.

## Dialogs, ajax, formatting, masking

| Ext JS | Bryntum | Note |
|---|---|---|
| `Ext.Msg.alert(title, msg)` | `await MessageDialog.alert({ title, message })` | `MessageDialog` is a singleton instance |
| `Ext.Msg.confirm(…)` | `await MessageDialog.confirm({ title, message })` | Resolves to `MessageDialog.okButton` / `.cancelButton` |
| `Ext.Msg.prompt(…)` | `await MessageDialog.prompt({ … })` | Resolves to `{ button, text }` |
| `Ext.toast('Saved')` | `Toast.show('Saved')` | Config form takes `html`, `timeout`, `showProgress` — not `type` |
| `Ext.Ajax.request({ url, success })` | `await AjaxHelper.get(url, { parseJson : true })` | Also `post(url, payload, options)`, `fetch`. Read `parsedJson` |
| `Ext.Date.format(d, 'Y-m-d')` | `DateHelper.format(date, 'YYYY-MM-DD')` | Unicode-style tokens. Also `parse`, `add`, `diff`, `as`, `startOf`, `endOf`, `clearTime` |
| `renderer : Ext.util.Format.currency` | NumberColumn `format : { style : 'currency', currency : 'USD' }` | A `NumberFormatConfig` |
| `Ext.util.Format.htmlEncode` | `StringHelper.encodeHtml` | Also `capitalize`, `hyphenate`, `stripHtmlTags`, `xss` tagged template |
| `Ext.Array` / `Ext.Object` | `ArrayHelper` / `ObjectHelper` | |
| `cmp.mask('Loading')` | `widget.masked = 'Loading'` | Or `Mask.mask(text, targetElement)` |
| `Ext.tip.ToolTip` | `Tooltip` widget, or a tooltip feature | See `bryntum-features` |

## Refreshing and selection

| Ext JS | Bryntum | Note |
|---|---|---|
| `grid.getView().refresh()` | `grid.refreshRows()` | Grid has no `refresh()` |
| refresh one column | `grid.refreshColumn(column)` | |
| `getSelectionModel().getSelection()` | `grid.selectedRecords` | Also `selectedRecord`, `selectedCells` |
| `sm.select(rec)` / `deselectAll()` | `grid.selectRow({ record })` / `deselectRow(record)` / `deselectAll()` | |
| `selModel : { mode : 'MULTI' }` | `selectionMode : { multiSelect : true, cell : false, checkbox : true }` | One config object, not a selection-model class |
