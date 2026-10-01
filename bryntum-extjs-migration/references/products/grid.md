# Grid

Support level: **moderate** (type B is mostly structural; type C — plain `Ext.grid.Panel` — follows the
mappings in `../api-mapping.md`, many tagged DOC: confirm them in the docs for the installed version).

Grid is a Panel in 7.x (`title`, `tools`, `tbar`, `bbar` work), so Ext header actions move to `tools`.

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

Rules:

- Render the Grid directly unless the old screen had sibling panels; then use a `Container` (`layout : 'hbox'`) with a
  `splitter`.
- `Ext.tree.Panel` → `TreeGrid` (or a Grid with a tree store) and a `type : 'tree'` column; data as nested `children`.
- Locked columns: `locked : true` on the column (Ext `lockable`/`locked` → same idea; the Grid creates the locked
  region).
- Grouped headers: parent column with `children : [...]`; collapsible groups use `collapsible`, `collapseMode :
  'toggleAll'`, `toggleAllHidden` (SRC).
- Computed grouping (`grouper.groupFn`) → a calculated field + `groupers : [{ field : '<calculatedField>' }]`.
- Group header text goes in the Group feature's `renderer({ groupRowFor, count, isFirstColumn })`. The per-column
  `groupRenderer` gets `{ groupRowFor, count, groupColumn, … }` and no `isFirstColumn` (`../patterns.md` §7).
- Ext `hideGroupedHeader` has no Group config: set `hidden : true` on the grouped column.
- `bufferedrenderer` and `infinite` configs are unnecessary — rendering is virtualized.
- Anything relying on custom Ext column xtypes, deep store pipelines or Ext-only plugins not in `../api-mapping.md`:
  report as unmapped rather than inventing a column type.

## Type B: Bryntum Grid inside an Ext shell

Mostly structural: remove `Ext.application`, `Viewport`, `Loader`, wrapper panels and controllers; copy the Bryntum
Grid config and normalize it for 7.x (`dataIndex` → `field`, header `items` → `tools`, `bind` → handlers).

## Limits

No official Ext → Bryntum Grid migration guide exists. Complete what the references support, and list the remaining
gaps in the report.
