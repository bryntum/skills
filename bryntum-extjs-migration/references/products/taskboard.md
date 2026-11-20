# TaskBoard (Ext `Kanban.*`)

Support level: **limited**. No official guide. Rows were name-checked against the 8.0.0 `@bryntum/taskboard-enterprise` package; unverified equivalences are tagged UNV.

Detect Bryntum's own Ext Kanban by `Kanban.*` classes (the namespace predates the product name), `xtype : 'kanban'` or
`Kanban.data.TaskStore`. Most "kanban" repos have none of these: see "Plain Ext boards" below, and
`SKILL.md` ("Judge by what the code does"): a repo named Kanban may need no Bryntum widget at all.

Tier: the Community tier covers the board, columns, `taskEdit`, `taskMenu`, `taskDrag`, `taskTooltip`. Swimlanes, `filterBar`,
`columnFilter`, `columnRename`, `columnResize`, `simpleTaskEdit` need the Core tier; `columnDrag`, `columnLock`, `columnSort`,
`swimlaneDrag` need the Enterprise tier. See `../api-mapping.md` §0.

| Ext (`Kanban.*`) | Bryntum TaskBoard | Tag |
|---|---|---|
| `Kanban.view.TaskBoard` | `new TaskBoard({ appendTo, ... })` | UNV |
| `taskStore`, `resourceStore` | `project : { tasks, resources, assignments }` (or `taskStore`/`resourceStore` configs on the project) | SRC |
| `Task.states : [...]` (columns come from the model) | explicit `columnField : 'status'` and `columns : [{ id, text, color }]`; the ids match the field values | SRC |
| `columnConfigs`, `fitColumns` | nothing to configure for `fitColumns`: by default columns are `flex : 1` with a 12em minimum (`--b-task-board-column-min-width`), so 3 columns share the board (467px each at 1400px) and it scrolls sideways only below 3 × the minimum. Fixed widths: `columns : [{ id, width : 300 }]` (scrolls when too narrow); `flex`, `width` and `minWidth` are the real column fields. `columnWidth`, `stretchColumns`, `columnMinWidth` aren't TaskBoard configs (`columnWidth : 'auto'` is silently ignored). `columnConfigs` has no equivalent name: put per-column config in `columns` | TEST |
| `editor`, `Kanban.editor.SimpleEditor` | `features : { taskEdit }`, `features : { simpleTaskEdit }` (Core) | SRC |
| `taskMenu`, `enableUserMenu` | `features : { taskMenu }` (on by default; right-click a card: `editTask`, `showTask`, `resources`, `column`, `removeTask`). `enableUserMenu` has no equivalent name | TEST (menu), UNV (`enableUserMenu`) |
| `dragZoneConfig` | `features : { taskDrag }` (config names in `TaskDragConfig`); `dragZoneConfig` has no equivalent name | SRC, UNV |
| `zoomLevel : 'mini'/'small'/'medium'/'large'`, `setZoomLevel` | `tasksPerRow` (1 to 4, default 1; the card size class follows: 1-2 large, 3 medium, 4 small) or a `{ type : 'zoomslider' }` toolbar widget. The old names probably map `mini` = 4, `small` = 3, `medium` = 2, `large` = 1 (unconfirmed: no Ext Kanban source to check against; `tasksPerRow` itself is tested) | TEST (`tasksPerRow`), UNV (name mapping) |
| `taskToolTpl`, `taskBodyTpl` (XTemplate) | `headerItems` / `bodyItems` / `footerItems` keyed items; item types `template`, `progress`, `image`, `rating`, `separator` (`collapse`, `chart` are Enterprise, `jsx`, `resourceAvatars`, `tags`, `todoList` Core), or `taskRenderer` | SRC |
| `Kanban.field.TaskFilter`, `ColumnFilter` | `{ type : 'taskfilterfield' }`, `{ type : 'columnfilterfield' }` | SRC |
| `Kanban.field.TaskHighlight` | none in the package: unmapped, build it custom | SRC (absent) |
| `Kanban.menu.UserPictureMenu` | the `resourceAvatars` item is in the default footer. To configure it use `footerItems : { resourceAvatars : {} }` (`true` throws `Cannot use 'in' operator`). It only displays initials/pictures: clicking an avatar opens no menu, so the Ext assign-a-user menu has no equivalent (the card's `taskMenu` has a `resources` item) | TEST |
| `isValidTransition()` | the `beforeTaskDrop` event: return `false` to veto and the card stays in its column. Args: `taskRecords` (array), `targetColumn` (`.id` is the column value), `targetSwimlane` (undefined without swimlanes), `targetTaskRecord`, `drag`, `domEvent` | TEST |

## Plain Ext boards (no `Kanban.*`)

The common real case: an `Ext.panel.Panel` (hbox) of column panels, one per status, each a dataview over a store
filtered by that status (columns generated from a status store through a ViewModel `bind`), cards from an
`Ext.XTemplate`, drag through `gridviewdragdrop` or `Ext.dd.DragZone/DropZone` that sets the record's state. Tested on one
such app (Vue): counts, card order and a drag matched the original.

| Plain Ext | TaskBoard |
|---|---|
| one column panel per status + a locally filtered store | `columnField : 'status'` + `columns : [{ id, text, color }]`. The column id and the task's `status` must be the same type (stringify both) |
| status store `add` / `remove` | `taskBoard.columns.data = [...]`, or `columns.add` / `columns.remove` |
| `XTemplate` card (`tpl`, `itemTpl`) | `headerItems` / `bodyItems` / `footerItems` keyed items, or `taskRenderer` |
| `DragZone` / `DropZone`, `gridviewdragdrop` + `record.set(state)` | the built-in `taskDrag`: nothing to write. `beforeTaskDrop` returning `false` vetoes |
| per-column store `sorters` | `taskSorterFn(a, b)` on the TaskBoard. Without it cards sort by `weight`, and a drag **sends `weight` in the sync request** (`{"updated":[{"status":"4004","weight":1400,"id":7000}]}`): a backend that rejects unknown fields must accept it. With `taskSorterFn` the drop's vertical position is ignored |
| dataview `selection` bind | `selectionChange` event |
| dataview `emptyText` | none seen (UNV) |
| single user reference (`ResourceId`, a nested `user` object) | an assignment record `{ id, event, resource }`; `task.resources[0]` gives the user |
| the backend's own envelope (several collections, nested `state`/`user`) | project `loadUrl` + `beforeLoadApply({ response })`: `data-migration.md` |

Defaults to turn off when the source lacked them: `taskEdit`, `taskMenu`, `columnHeaderMenu` (the "…" button) and
`columnToolbars` (a "+" under every column), in Vue `taskEditFeature={false}` etc. (names from `BryntumTaskBoardProps`; not
checked at the Community tier). Features are `xFeature` props in the wrappers (`SKILL.md`). `footerItems : { resourceAvatars : null }` removes the default footer.

**Card color.** `eventColor` on a task (or `color` on a column) sets `--b-primary` and the `b-colorize` class, but the card
chrome doesn't use it: nothing shows. A priority stripe needs a rule such as
`.b-task-board-card.b-colorize { --b-task-board-card-border-inline-start : 10px solid var(--b-primary); }` (the variables are
`--b-task-board-card-*` in `taskboard.css`). Named colors are the Bryntum palette (`'lime'` is yellow-green, not CSS lime):
use a hex string when the exact color matters.

**CSS on a mixed page** (TaskBoard + Grid): `core.thin.css`, `grid.thin.css`, `taskboard.thin.css`, then one theme from either
package (`@bryntum/taskboard-<tier>/svalbard.css`). TaskBoard isn't built on Grid, so there is no "highest product".
Every Bryntum widget draws its own trial badge, so five widgets show five badges; one can cover a toolbar and block a
click in a test.

Rules:

- **Single assignment becomes an assignment record.** Legacy tasks carry `ResourceId`. Bryntum TaskBoard stores
  assignments separately: create one assignment record per non-empty `ResourceId`. Count tasks and assignments before and
  after. (Same rule as `../data-migration.md` for other products.)
- Assignment records use the fields `event` and `resource` (not `taskId`/`resourceId`); wrong names leave `task.resources` empty with no error.
- TaskBoard has **no time axis**: tasks need no dates.
- Task field names that a card shows need declared model fields (or a column/item `field` on 8.0 for column-like items).
