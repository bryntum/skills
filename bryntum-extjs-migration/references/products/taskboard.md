# TaskBoard (Ext `Kanban.*`)

Support level: **limited**. No official guide. Rows come from the suite's Ext migration knowledge base and were
name-checked against the 8.0.0 `@bryntum/taskboard-enterprise` package; unverified equivalences are tagged UNV.

Detect it by `Kanban.*` classes (the namespace predates the product name), `xtype : 'kanban'` or `Kanban.data.TaskStore`.

Tier: Community covers the board, columns, `taskEdit`, `taskMenu`, `taskDrag`, `taskTooltip`. Swimlanes, `filterBar`,
`columnFilter`, `columnRename`, `columnResize`, `simpleTaskEdit` need Business; `columnDrag`, `columnLock`, `columnSort`,
`swimlaneDrag` need Enterprise. See `../api-mapping.md` §0.

| Ext (`Kanban.*`) | Bryntum TaskBoard | Tag |
|---|---|---|
| `Kanban.view.TaskBoard` | `new TaskBoard({ appendTo, ... })` | UNV |
| `taskStore`, `resourceStore` | `project : { tasks, resources, assignments }` (or `taskStore`/`resourceStore` configs on the project) | SRC |
| `Task.states : [...]` (columns come from the model) | explicit `columnField : 'status'` and `columns : [{ id, text, color }]`; the ids match the field values | SRC |
| `columnConfigs`, `fitColumns` | per-column config in `columns`; `columnWidth` | UNV |
| `editor`, `Kanban.editor.SimpleEditor` | `features : { taskEdit }`, `features : { simpleTaskEdit }` (Business) | SRC |
| `taskMenu`, `enableUserMenu` | `features : { taskMenu }` | UNV |
| `dragZoneConfig` | `features : { taskDrag }` | UNV |
| `zoomLevel : 'mini'/'small'/'medium'/'large'`, `setZoomLevel` | `tasksPerRow` config (4/3/2/1, UNV mapping of the old names) or a `{ type : 'zoomslider' }` toolbar widget | SRC |
| `taskToolTpl`, `taskBodyTpl` (XTemplate) | `headerItems` / `bodyItems` / `footerItems` keyed items; item types `template`, `progress`, `image`, `rating`, `separator` (`collapse`, `chart` are Enterprise, `jsx`, `resourceAvatars`, `tags`, `todoList` Business), or `taskRenderer` | SRC |
| `Kanban.field.TaskFilter`, `ColumnFilter` | `{ type : 'taskfilterfield' }`, `{ type : 'columnfilterfield' }` | SRC |
| `Kanban.field.TaskHighlight` | none: custom | UNV |
| `Kanban.menu.UserPictureMenu` | `footerItems : { resourceAvatars : true }` | UNV |
| `isValidTransition()` | the `beforeTaskDrop` event (return `false` to veto; check its context shape in the docs) | SRC (event name), UNV (equivalence) |

Rules:

- **Single assignment becomes an assignment record.** Legacy tasks carry `ResourceId`. Bryntum TaskBoard stores
  assignments separately: create one assignment record per non-empty `ResourceId`. Count tasks and assignments before and
  after. (Same rule as `../data-migration.md` for other products.)
- TaskBoard has **no time axis**: tasks need no dates.
- Task field names that a card shows need declared model fields (or a column/item `field` on 8.0 for column-like items).
