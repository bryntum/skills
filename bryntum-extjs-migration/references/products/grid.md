# Grid

Support level: **moderate** (type B is mostly structural; type C — plain `Ext.grid.Panel` — follows the
mappings in `../api-mapping.md`, many tagged DOC: confirm them in the docs for the installed version).

Grid is a Panel (`title`, `tools`, `tbar`, `bbar` work), so Ext header actions move to `tools`.

## Type C: `Ext.grid.Panel` → `Grid`

Typical result for an `Ext.grid.Panel` with an ajax proxy (`rootProperty : 'data'`), date/number/check columns,
`cellediting`, `gridfilters` and the `grouping` feature:

```js
new Grid({
    appendTo : 'app',
    title    : 'Orders',
    store    : { modelClass : Order, readUrl : '/api/orders', responseDataProperty : 'data', autoLoad : true },
    columns  : [
        { text : 'Customer', field : 'customer', flex : 1 },
        { type : 'date',   text : 'Date',  field : 'date',  format : 'YYYY-MM-DD' },
        { type : 'number', text : 'Total', field : 'total' },       // number formatting: verify `format` options
        { type : 'check',  text : 'Paid',  field : 'paid' }
    ],
    features : {
        cellEdit : true,          // default on
        filter   : true,          // or filterBar : true for a filter row
        group    : 'status'
    }
});
```

That is the vanilla form. In React, Vue and Angular each feature is its own prop or input (`cellEditFeature`, `filterFeature`,
`groupFeature`; see `SKILL.md`, Converting).

Rules:

- Render the Grid directly unless the old screen had sibling panels; then use a `Container` (`layout : 'hbox'`) with a
  `splitter`.
- `Ext.tree.Panel` → `TreeGrid` (or a Grid with a tree store) and a `type : 'tree'` column; data as nested `children`.
  Ext toggles a parent on double-click; Bryntum only from the expander, so port it as `cellDblClick` + `toggleCollapse`
  and turn `cellEdit` off (`../idioms.md`). Tree drag and drop (re-parent, between two trees): `rowReorder` with `dropOnLeaf`
  and `allowCrossGridDrag` (`../api-mapping.md` §5). Tree persistence caveats (a same-parent reorder is not in
  `store.changes`): `../idioms.md`.
  - **Visible root** (`rootVisible : true`, a root row above the tree): Bryntum has no root row, so a port that just drops it loses
    "add a sub node to the root" (1 of 3 E3 runs), and the 3 runs treated it 3 different ways. Decide once and say so in the report:
    load the root as a real top-level record (`[{ id : 'root', name : 'Root', expanded : true, children : [...] }]`) and keep it, or drop it
    and add a top-level "Add" action.
  - **Drop onto a parent**: Bryntum's `rowReorder` inserts the node as the parent's **first** child (`RowReorder.js`:
    `parent.insertChild(records, parent.children?.[0])`); Ext appended it **last** (1 of 3 E3 runs got the order wrong). To keep Ext's
    order, move the record in `gridRowDrop` (`parent.appendChild(record)`, UNV) and test a drop onto a parent with children.
  - **Order field**: don't name a field `index` (the Store writes `record.index`; the TreeNode mixin owns `parentIndex`/`orderedParentIndex`):
    call it `sortIndex`, or the model works in a test and breaks later.
- Ext validation (`regex`, `maskRe`, `vtype`), form load/reset/dirty and `grid.setDisabled`: `../api-mapping.md` §2 and §7,
  `../patterns.md` §6.
- Toolbar widths: five text+icon buttons that fit a 350 px Ext tree toolbar overflowed into the menu icon with the Stockholm theme; even
  the compact recipe (`styling.md`) needed about 520 px. Budget about 100 px per button (measured with the Stockholm theme; font and padding differ per theme).
- Locked columns: `locked : true` on the column (Ext `lockable`/`locked` → same idea; the Grid creates the locked
  region).
- Grouped headers: parent column with `children : [...]`; collapsible groups use `collapsible`, `collapseMode :
  'toggleAll'`, `toggleAllHidden` (SRC).
- Computed grouping (`grouper.groupFn`) → a calculated field + `groupers : [{ field : '<calculatedField>' }]`.
- Group header text goes in the Group feature's `renderer({ groupRowFor, count, isFirstColumn })`. The per-column
  `groupRenderer` gets `{ groupRowFor, count, groupColumn, … }` and no `isFirstColumn` (`../patterns.md` §7).
  `groupRowFor` is `'!!novalue!!'` for records with an empty group value: map it to `''` in a custom renderer (SRC).
- Ext `hideGroupedHeader` has no Group config: set `hidden : true` on the grouped column.
- Header menu: Ext shows Sort Ascending/Descending + a Columns submenu (+ grouping items); `headerMenu` adds more.
  Match it with `features : { headerMenu : { items : { hideColumn : false, resizeColumn : false, multiSort : false,
  sortAsc : { text : '…' } } } }` (keys also `columnPicker`, `sortDesc`, `groupAsc`, `groupDesc`, `groupRemove`;
  per-column extras via `headerMenuItems`). The Columns submenu lists `column.text`, so an icon-only column needs a
  `text` plus `icon` and `headerRenderer : () => ''` to keep the header blank, or `hideable : false` (SRC).
- `bufferedrenderer` and `infinite` configs are unnecessary — rendering is virtualized.
- Anything relying on custom Ext column xtypes, deep store pipelines or Ext-only plugins not in `../api-mapping.md`:
  report as unmapped rather than inventing a column type.

## Type B: Bryntum Grid inside an Ext shell

Mostly structural: remove `Ext.application`, `Viewport`, `Loader`, wrapper panels and controllers; copy the Bryntum
Grid config and normalize it for 8.0 (`dataIndex` → `field`, header `items` → `tools`, `bind` → handlers).

**UMD globals.** A type B app that reads `bryntum.<product>.X` (e.g. `bryntum.grid.DataGenerator`, `bryntum.grid.Column`)
becomes an import from the package that owns `X`: `DataGenerator`, `Store`, `DateHelper` come from `@bryntum/core-<tier>`,
`Grid`, `Column` from `@bryntum/grid-<tier>`. Check `node_modules/@bryntum/*/entry.js`.

## Limits

No official Ext → Bryntum Grid migration guide exists. Complete what the references support, and list the remaining
gaps in the report.

## Server contracts for plain grids

Tested with two Ext 4 CRUD apps (PHP, Ext Direct): reads keep the server's JSON (`readUrl` + `responseDataProperty`), but writes
don't (`../data-migration.md`: flat bodies, `{ data : [...] }`, `commit()` override, `AjaxHelper` failure semantics). Ext's
paging (seen in a React port): `pageStartParamName` and `pageParamName` are mutually exclusive on the store (setting both throws at runtime); `loadPage()` takes two arguments in the typings (`loadPage(page, {})`); the paging summary text comes from the locale (`LocaleHelper.publishLocale`). Ext's
selection behavior matched with `selectionMode : { multiSelect : false }`: arrows move and select, Ctrl+click replaces the
selection, Ctrl+A and Shift+click don't multi-select, Delete does nothing.
