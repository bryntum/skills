# Editing, tooltips, undo, export

Tier marks are 8.x; an unmarked feature is in every tier.

## Editors and tooltips

- Editor popup: `eventEdit` (Scheduler Community and Business, Calendar; on) / `taskEdit`
  (Scheduler Enterprise, on there; Gantt Business and Enterprise, on; TaskBoard, on). Gantt
  Community has the simpler `basicTaskEdit` as its default (it exists in every tier). For adding,
  removing or replacing fields and tabs, load the `bryntum-editor` skill.
- Lighter inline editor (name field only): `simpleEventEdit` (Scheduler, Business, off).
- Bar tooltip: the feature name differs by product.
  - Scheduler: `eventTooltip : { template : ({ eventRecord }) => … }`.
  - Gantt: `taskTooltip : { template : ({ taskRecord, startClockHtml, endClockHtml }) => … }`. Gantt
    has no `eventTooltip`; on 8.x the unknown key throws `Invalid type name`.
  - Calendar: `eventTooltip` with `renderer` / `titleRenderer`.
- End dates are exclusive: with 24-hour working days, a task that finishes Friday has `endDate`
  at Saturday 00:00. With an 08:00–17:00 calendar it ends Friday 17:00, so subtracting a day would
  show Thursday. Use `new Date(endDate - 1)` (1 ms) for an inclusive end date in a custom template.
- Grid cell tooltip: `cellTooltip` (Business, off),
  `cellTooltip : { tooltipRenderer : ({ record }) => record.name }`.

## Undo / redo

Undo is Business and up: the `StateTrackingManager` and the `UndoRedo` widget are Business,
TaskBoard's undo widget is Enterprise. It needs two independent settings on the project's
`StateTrackingManager`:

- **Recording:** `project : { stm : { autoRecord : true } }`. Without it, edits aren't captured
  even when the STM is enabled.
- **Enabling:** the STM starts disabled, and `autoRecord` doesn't change that. Either add an
  `UndoRedo` widget, which enables it once the project has loaded, or call `project.stm.enable()`
  after `await project.commitAsync()`. Enabling at construction records the initial data load as
  undo steps; neither of these does.

Verified headless in Gantt 7.3.7: the widget alone, `autoRecord` alone, and `enable()` alone each
leave `canUndo` `false` after an edit. `autoRecord` plus either the widget or `enable()` makes the
edit undoable. Then call `project.stm.undo()` / `.redo()`, or let the widget do it.

For your own undo/redo buttons, refresh `disabled` from `stm.canUndo` / `stm.canRedo` on the STM
events `recordingStop`, `restoringStop`, `queueReset` and `disabled` (the built-in widget's
list). Gantt disables the STM while it recalculates after an undo, so `canUndo` is still `false`
at `restoringStop`; the correct value only appears on the `disabled` event that follows.

`UndoRedo` widget types: `{ type : 'undoredo' }` for Scheduler and Gantt, and
`{ type : 'taskboardundoredo' }` for TaskBoard (Enterprise). On 8.x a Gantt gets the `undoredo`
widget from the Scheduler package — add `import '@bryntum/scheduler-<tier>';` or the toolbar item
fails with `Invalid type name`. Useful configs: `text : true` (button
labels), `showZeroActionBadge`, and `items : { transactionsCombo : null }` (hide the history combo).
The buttons render with `data-ref="undoBtn"` / `data-ref="redoBtn"`. Target those in tests, since
the Redo button's accessible name contains "undone" and matches `/undo/i`.

A plain Grid has no project and no `UndoRedo` widget, so wire the manager yourself:

```javascript
import { StateTrackingManager } from '@bryntum/core-<tier>';   // Business or Enterprise

const stm = new StateTrackingManager({ autoRecord : true });
stm.addStore(grid.store);
stm.enable();          // nothing is recorded until this is called
```

## Getting data out

All of these are Enterprise.

| Output | Feature | Call | Gotcha |
|---|---|---|---|
| Excel | `excelExporter` (experimental, off) | `grid.features.excelExporter.export({ filename : 'data' })` | See below. Gantt: registers only after `import '@bryntum/grid-<tier>';` |
| PDF | `pdfExport` (off) | `await grid.features.pdfExport.export()` | Needs a running export server (`bryntum/pdf-export-server`) via `exportServer`; no client-only path |
| Print | `print` (off) | `grid.print({})` | Called on the component, not `grid.features.print.print()`. The config argument is typed as required, so pass `{}` in TypeScript. No server |
| MS Project | `mspExport` (Gantt, off) | `gantt.features.mspExport.export()` | Produces MS Project XML; no server |
| Print a Calendar | Calendar `print` | `calendar.print()` | Calendar has no PDF-export feature |

**Excel setup.** The default `xlsProvider` (`WriteExcelFileProvider`) calls
`globalThis.writeXlsxFile` and expects it to return a `Blob`. Leave `xlsProvider` alone unless
you have a custom provider class with a static `write` method; passing the library function
there throws `"xlsProvider" library is required`. Use `write-excel-file@3`. It has no root
export, so import the browser entry:

```javascript
import writeXlsxFile from 'write-excel-file/browser';
globalThis.writeXlsxFile = writeXlsxFile;   // TypeScript: (globalThis as any).writeXlsxFile
```

**Excel output for Scheduler.** One row per event: the grid's own columns, then
Task / Starts / Ends. Dates are written as date-only cells (`yyyy-mm-dd`), so times disappear. For
intra-day data, set `dateFormat : 'YYYY-MM-DD HH:mm'`, which writes the dates as formatted
strings. Replace the event columns with
`exporterConfig : { eventColumns : [{ text : 'Appointment', field : 'name' }] }`.

These preserve columns, grouping and styling, so prefer them over serializing `store.records`
unless the user asked for CSV.
