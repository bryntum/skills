# TaskBoard (Ext `Kanban.*`)

Support level: **limited**. No official guide. Rows come from the suite's Ext migration knowledge base and were
name-checked against the 8.0.0 `@bryntum/taskboard-enterprise` package; unverified equivalences are tagged UNV.

Detect it by `Kanban.*` classes (the namespace predates the product name), `xtype : 'kanban'` or `Kanban.data.TaskStore`.

Tier: the Community tier covers the board, columns, `taskEdit`, `taskMenu`, `taskDrag`, `taskTooltip`. Swimlanes, `filterBar`,
`columnFilter`, `columnRename`, `columnResize`, `simpleTaskEdit` need the Core tier; `columnDrag`, `columnLock`, `columnSort`,
`swimlaneDrag` need the Enterprise tier. See `../api-mapping.md` §0.

| Ext (`Kanban.*`) | Bryntum TaskBoard | Tag |
|---|---|---|
| `Kanban.view.TaskBoard` | `new TaskBoard({ appendTo, ... })` | UNV |
| `taskStore`, `resourceStore` | `project : { tasks, resources, assignments }` (or `taskStore`/`resourceStore` configs on the project) | SRC |
| `Task.states : [...]` (columns come from the model) | explicit `columnField : 'status'` and `columns : [{ id, text, color }]`; the ids match the field values | SRC |
| `columnConfigs`, `fitColumns` | nothing to configure for `fitColumns`: by default columns are `flex : 1` with a 12em minimum (`--b-task-board-column-min-width`), so 3 columns share the board (467px each at 1400px) and it scrolls sideways only below 3 × the minimum. Fixed widths: `columns : [{ id, width : 300 }]` (scrolls when too narrow); `flex`, `width` and `minWidth` are the real column fields. `columnWidth`, `stretchColumns`, `columnMinWidth` aren't TaskBoard configs (`columnWidth : 'auto'`, claimed in the Ext knowledge base, is silently ignored). `columnConfigs` has no equivalent name: put per-column config in `columns` | TEST |
| `editor`, `Kanban.editor.SimpleEditor` | `features : { taskEdit }`, `features : { simpleTaskEdit }` (Core) | SRC |
| `taskMenu`, `enableUserMenu` | `features : { taskMenu }` (on by default; right-click a card: `editTask`, `showTask`, `resources`, `column`, `removeTask`). `enableUserMenu` has no equivalent name | TEST (menu), UNV (`enableUserMenu`) |
| `dragZoneConfig` | `features : { taskDrag }` (config names in `TaskDragConfig`); `dragZoneConfig` has no equivalent name | SRC, UNV |
| `zoomLevel : 'mini'/'small'/'medium'/'large'`, `setZoomLevel` | `tasksPerRow` (1 to 4, default 1; the card size class follows: 1-2 large, 3 medium, 4 small) or a `{ type : 'zoomslider' }` toolbar widget. The old names map `mini` = 4, `small` = 3, `medium` = 2, `large` = 1 per the suite's Ext knowledge base (no Ext Kanban source was available to confirm; `tasksPerRow` itself is tested) | TEST (`tasksPerRow`), KB (name mapping) |
| `taskToolTpl`, `taskBodyTpl` (XTemplate) | `headerItems` / `bodyItems` / `footerItems` keyed items; item types `template`, `progress`, `image`, `rating`, `separator` (`collapse`, `chart` are Enterprise, `jsx`, `resourceAvatars`, `tags`, `todoList` Core), or `taskRenderer` | SRC |
| `Kanban.field.TaskFilter`, `ColumnFilter` | `{ type : 'taskfilterfield' }`, `{ type : 'columnfilterfield' }` | SRC |
| `Kanban.field.TaskHighlight` | none in the package: unmapped, build it custom | SRC (absent) |
| `Kanban.menu.UserPictureMenu` | the `resourceAvatars` item is in the default footer. To configure it use `footerItems : { resourceAvatars : {} }` (`true` throws `Cannot use 'in' operator`). It only displays initials/pictures: clicking an avatar opens no menu, so the Ext assign-a-user menu has no equivalent (the card's `taskMenu` has a `resources` item) | TEST |
| `isValidTransition()` | the `beforeTaskDrop` event: return `false` to veto and the card stays in its column. Args: `taskRecords` (array), `targetColumn` (`.id` is the column value), `targetSwimlane` (undefined without swimlanes), `targetTaskRecord`, `drag`, `domEvent` | TEST |

Rules:

- **Single assignment becomes an assignment record.** Legacy tasks carry `ResourceId`. Bryntum TaskBoard stores
  assignments separately: create one assignment record per non-empty `ResourceId`. Count tasks and assignments before and
  after. (Same rule as `../data-migration.md` for other products.)
- Assignment records use the fields `event` and `resource` (not `taskId`/`resourceId`); wrong names leave `task.resources` empty with no error.
- TaskBoard has **no time axis**: tasks need no dates.
- Task field names that a card shows need declared model fields (or a column/item `field` on 8.0 for column-like items).
