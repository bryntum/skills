# Grid features (also apply to Scheduler, Gantt and other grid-based views)

## Highlighting

- One row: set the record's `cls` field (`record.cls = 'is-late'`).
- One column's cells: column `cellCls` (static), or a `renderer` that sets `cellElement.classList`.
- Alternating rows: `stripe` feature. `:nth-child(even)` CSS breaks under row virtualization.
- Reading the selection: `grid.selectedRecords` / `selectedRecord` / `selectedCells`.

## Trees

- `TreeGrid`, or `Grid` + `tree` feature, with exactly one `{ type : 'tree' }` column and
  `children` arrays in the data.
- `treeGroup` turns flat rows into a tree, one level per field:
  `features : { treeGroup : { levels : ['country', 'city'] } }`.
- Expand / collapse from code through the feature, not the record:
  `await grid.features.tree.expandAll()`, `.collapseAll()`, `.expandTo(record)`.

## Grouping and totals

| The user asks for | Use | Snippet |
|---|---|---|
| Group rows | `group` (on by default for Grid and Scheduler, off for TreeGrid) | `features : { group : 'city' }` or `{ group : { field : 'city', ascending : false } }` |
| A totals row | `summary` (off) + column `sum` | `features : { summary : true }`, `{ field : 'score', sum : 'sum' }` |
| Per-group totals | `groupSummary` (off) + column `sum` | `features : { group : 'city', groupSummary : true }` |
| Several totals in one column | column `summaries` (replaces `sum` + `summaryRenderer`) | `summaries : [{ sum : 'sum' }, { sum : 'average' }]` |
| Format a total | column `summaryRenderer` | `summaryRenderer : ({ sum }) => \`Total: ${sum}\`` |

The grid UI doesn't support grouping by more than one field (the multi-grouper store API is
internal). Use `treeGroup` for a multi-level breakdown.

## Filtering, searching, sorting

- Header filter UI: `filter` (menu + popup) or `filterBar` (inline row of fields), both off by default.
- From code: `store.filter({ property : 'city', value : 'Paris', operator : '=' })`, undone with
  `store.clearFilters()`. Replacing `store.data` with a filtered array loses the original rows.
- A search box: `search` has the logic but no UI — supply the field and call
  `grid.features.search.search('paris')`.
- Type-to-find in the focused column: `quickFind`.
- Multi-column sort: `store.sort('name', true, true)` — the third argument appends a sorter.
