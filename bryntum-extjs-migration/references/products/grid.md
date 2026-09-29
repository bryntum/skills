# Grid

Support level: **moderate** (type B verified in finished examples; type C — plain `Ext.grid.Panel` — follows the
mappings in `../api-mapping.md`, many tagged DOC: confirm them in the docs for the installed version).

Finished examples (extjs-migration-agent repo): `grid-extjsmodern-vite` (wrapper removal, header items → `tools`,
ViewModel binds → `selectionChange`, `groupRenderer` 7.x args), `grid-extjs-groupedheaders-vite` (grouped and
collapsible headers, template/date/percent/check columns, combo editor, `StringHelper.xss`).

## What a Bryntum Grid app looks like

- `new Grid({ appendTo, columns, data | store, features, tbar, tools })` — Grid is a Panel (`title`, `tools`, `tbar`,
  `bbar` work)
- columns are plain configs `{ text, field, width | flex, type, editor, locked, renderer }`
- behavior is `features : { ... }`, not plugins
- header actions live in `tools`
- ViewModel state becomes direct widget state plus handlers (`onSelectionChange`, field `onChange`) or framework state

## Type C: `Ext.grid.Panel` → `Grid`

Typical conversion:

```js
// Ext
Ext.create('Ext.grid.Panel', {
    title   : 'Orders',
    store   : { model : 'Order', proxy : { type : 'ajax', url : '/api/orders', reader : { rootProperty : 'data' } }, autoLoad : true },
    columns : [
        { text : 'Customer', dataIndex : 'customer', flex : 1 },
        { xtype : 'datecolumn', text : 'Date', dataIndex : 'date', format : 'Y-m-d' },
        { xtype : 'numbercolumn', text : 'Total', dataIndex : 'total', format : '0.00' },
        { xtype : 'checkcolumn', text : 'Paid', dataIndex : 'paid' }
    ],
    plugins  : { cellediting : { clicksToEdit : 1 }, gridfilters : true },
    features : [{ ftype : 'grouping', groupHeaderTpl : '{name}' }],
    renderTo : Ext.getBody()
});

// Bryntum
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
  'toggleAll'`, `toggleAllHidden` (SRC, groupedheaders example).
- Computed grouping (`grouper.groupFn`) → a calculated field + `groupers : [{ field : '<calculatedField>' }]`.
- Group header renderers use the 7.x argument shape `groupRenderer({ groupColumn, groupRowFor, isFirstColumn })`.
- `bufferedrenderer` and `infinite` configs are unnecessary — rendering is virtualized.
- Anything relying on custom Ext column xtypes, deep store pipelines or Ext-only plugins not in `../api-mapping.md`:
  report as unmapped rather than inventing a column type.

## Type B: Bryntum Grid inside an Ext shell

Mostly structural: remove `Ext.application`, `Viewport`, `Loader`, wrapper panels and controllers; copy the Bryntum
Grid config and normalize it for 7.x (`dataIndex` → `field`, header `items` → `tools`, `bind` → handlers).

## Limits

No official Ext → Bryntum Grid migration guide exists. Complete what the references support, and list the remaining
gaps in the report.
