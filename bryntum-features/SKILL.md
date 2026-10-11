---
name: bryntum-features
description: >
  Find the built-in Bryntum feature for what the user is describing before writing a custom
  renderer, custom CSS, or a hand-drawn overlay. Use alongside the `bryntum` skill whenever a
  request is phrased as an outcome rather than an API — "shade 11:00-13:00 as lunch", "grey out
  weekends", "working hours are 9 to 5", "mark today", "draw arrows between tasks", "group by
  city", "a totals row", "a search box", "export to Excel", "print", "undo", "swimlanes",
  "baselines", "critical path", "recurring events", "split a task", "WIP limit", "highlight
  that row". Most of these are one config.
metadata:
  tags: bryntum, features, scheduler, schedulerpro, gantt, grid, calendar, taskboard, timeranges, nonworkingtime, export, undo
---

# Bryntum features

Most outcome-style requests map to a single entry in the `features` config. The usual mistake
is not knowing the feature exists and hand-building it with an `eventRenderer`, a `cellCls`,
positioned `<div>`s or CSS. That can look right while missing virtualization, export, state, or
(for working hours) the scheduling engine entirely. Check the index first; the reference files
have snippets and per-product differences.

Verified against Bryntum 7.3.6 source.

## How features behave

- The key is the class name with a lowercase first letter: `GroupSummary` → `groupSummary`,
  `CriticalPaths` → `criticalPaths`. `true` enables with defaults, an object configures, `false`
  turns off one that is on by default.
- Configure features at construction. A feature added to `features` after the component is built
  never runs its paint-time init. To switch one on later, include it in the config and flip
  `widget.features.x.disabled = false`.
- The React and Vue wrappers take each feature as a `<name>Feature` prop (`nonWorkingTimeFeature`,
  `eventTooltipFeature={{ … }}`), not a `features` object. Runtime access is still
  `instance.features.<name>`.
- Registered on by default is not the same as active: Gantt's `criticalPaths` ships
  `disabled : true`.
- The same concept can have different names per product: Scheduler `eventEdit` vs Gantt
  `taskEdit`; Scheduler `eventTooltip` vs Gantt `taskTooltip`; `template` in Scheduler and Gantt
  tooltips vs `renderer` in Calendar's.

## Index

| The user asks for | Use | Reference |
|---|---|---|
| A shaded band across the timeline (lunch, maintenance window) | `timeRanges` + `project.timeRanges` | [timeline](references/timeline.md) |
| A band for one resource only (PTO) | `resourceTimeRanges` | [timeline](references/timeline.md) |
| A "now" line | `timeRanges : { showCurrentTimeLine : true }` | [timeline](references/timeline.md) |
| Grey out weekends / off-hours | `nonWorkingTime`, `eventNonWorkingTime`, `resourceNonWorkingTime`, `taskNonWorkingTime` — pick by where the grey goes | [timeline](references/timeline.md) |
| Working hours that affect scheduling (Pro / Gantt) | Project `calendars` | [timeline](references/timeline.md) |
| Temporarily highlight a span from code | `timeSpanHighlight` (Pro / Gantt) | [timeline](references/timeline.md) |
| Arrows between events / tasks | `dependencies` (+ `dependencyEdit`) | [timeline](references/timeline.md) |
| Events inside a parent bar; splitting an event | `nestedEvents`; `eventSegments` | [timeline](references/timeline.md) |
| Highlight a row, a column, stripes | record `cls`, column `cellCls`, `stripe` | [grid](references/grid.md) |
| Tree, or a tree built from flat rows | `TreeGrid` / `tree`; `treeGroup` | [grid](references/grid.md) |
| Group rows, totals row, per-group totals | `group`, `summary`, `groupSummary` | [grid](references/grid.md) |
| Filter UI, search box, type-to-find, multi-sort | `filter` / `filterBar`, `search`, `quickFind`, `store.sort(…, true)` | [grid](references/grid.md) |
| Event/task editor, tooltips | `eventEdit` / `taskEdit`, `eventTooltip` / `taskTooltip` (Gantt), `cellTooltip` | [editing & export](references/editing-and-export.md) |
| Undo / redo | Project `stm : { autoRecord : true }` plus an `UndoRedo` widget or `stm.enable()` — both halves are needed | [editing & export](references/editing-and-export.md) |
| Excel, PDF, print, MS Project | `excelExporter`, `pdfExport`, `print`, `mspExport` | [editing & export](references/editing-and-export.md) |
| Critical path, baselines, progress line, rollups, deadlines | Gantt features | [gantt](references/gantt.md) |
| Resource load / utilisation | `ResourceHistogram` / `ResourceUtilization` views | [gantt](references/gantt.md) |
| Calendar views, recurring events, hiding weekends | `modes`, `recurrenceRule`, `hideNonWorkingDays` | [calendar & taskboard](references/calendar-taskboard.md) |
| Swimlanes, card content, WIP limits | `swimlaneField`, `bodyItems`, (no WIP config) | [calendar & taskboard](references/calendar-taskboard.md) |
| Drag from a grid or sidebar onto the timeline | `bryntum-drag-and-drop` skill | — |

## Things that do not exist

These are the most commonly invented APIs in this area:

- TaskBoard WIP limits — no config; build it with `beforeTaskDrop` and say it is custom.
- `timeRangesStore` / `resourceTimeRangesStore` — the stores are singular: `timeRangeStore`,
  `resourceTimeRangeStore`.
- A `taskFilter` TaskBoard feature — it is `columnFilter` or `filterBar`.
- `Grid#refresh()` — Grid has `refreshRows()` and `refreshColumn()`. Calendar, TaskBoard,
  Scheduler and Gantt do have `refresh()`.
- Multi-field grouping in the grid UI — use `treeGroup`.
- A Grid `UndoRedo` widget.
- `eventTooltip` in Gantt — it is `taskTooltip`, and the unknown key is silently ignored.
- Client-only PDF export — `pdfExport` needs the export server.
- `columnLines` driven by time ranges — it draws from ViewPreset ticks only.

## Related

- `bryntum-editor` — customizing the event/task editor.
- `bryntum-styling` — event bar content and `eventRenderer`.
- `bryntum-drag-and-drop` — dragging onto the timeline from outside.
- `bryntum-from-extjs` — Ext JS idioms that do not apply here.
