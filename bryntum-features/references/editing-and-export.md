# Editing, tooltips, undo, export

## Editors and tooltips

- Editor popup: `eventEdit` (Scheduler, on) / `taskEdit` (Scheduler Pro + Gantt, on). For adding,
  removing or replacing fields and tabs, load the `bryntum-editor` skill.
- Lighter inline editor (name field only): `simpleEventEdit` (Scheduler, off).
- Bar tooltip: the feature name differs by product.
  - Scheduler / Scheduler Pro: `eventTooltip : { template : ({ eventRecord }) => … }`.
  - Gantt: `taskTooltip : { template : ({ taskRecord, startClockHtml, endClockHtml }) => … }`. Gantt
    has no `eventTooltip`, and an unknown feature key is silently ignored.
  - Calendar: `eventTooltip` with `renderer` / `titleRenderer`.
- End dates are exclusive: with 24-hour working days, a task that finishes Friday has `endDate`
  at Saturday 00:00. With an 08:00–17:00 calendar it ends Friday 17:00, so subtracting a day would
  show Thursday. Use `new Date(endDate - 1)` (1 ms) for an inclusive end date in a custom template.
- Grid cell tooltip: `cellTooltip` (off),
  `cellTooltip : { tooltipRenderer : ({ record }) => record.name }`.

## Undo / redo

Undo needs two independent settings on the project's `StateTrackingManager`:

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

`UndoRedo` widget types: `{ type : 'undoredo' }` for Scheduler, Scheduler Pro and Gantt, and
`{ type : 'taskboardundoredo' }` for TaskBoard. Useful configs: `text : true` (button labels),
`showZeroActionBadge`, and `items : { transactionsCombo : null }` (hide the history combo). The
buttons render with `data-ref="undoBtn"` / `data-ref="redoBtn"`. Target those in tests, since the
Redo button's accessible name contains "undone" and matches `/undo/i`.

A plain Grid has no project and no `UndoRedo` widget, so wire the manager yourself:

```javascript
import { StateTrackingManager } from '@bryntum/grid';

const stm = new StateTrackingManager({ autoRecord : true });
stm.addStore(grid.store);
stm.enable();          // nothing is recorded until this is called
```

## Getting data out

| Output | Feature | Call | Gotcha |
|---|---|---|---|
| Excel | `excelExporter` (experimental, off) | `grid.features.excelExporter.export({ filename : 'data' })` | See below |
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

**Excel output for Scheduler / Scheduler Pro.** One row per event: the grid's own columns, then
Task / Starts / Ends. Dates are written as date-only cells (`yyyy-mm-dd`), so times disappear. For
intra-day data, set `dateFormat : 'YYYY-MM-DD HH:mm'`, which writes the dates as formatted
strings. Replace the event columns with
`exporterConfig : { eventColumns : [{ text : 'Appointment', field : 'name' }] }`.

These preserve columns, grouping and styling, so prefer them over serializing `store.records`
unless the user asked for CSV.
