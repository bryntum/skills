---
name: bryntum-features
description: >
  Find the built-in Bryntum feature for what the user is describing before writing a custom
  renderer, custom CSS, or a hand-drawn overlay, and the tier it needs. Use alongside the
  `bryntum` skill whenever a request is phrased as an outcome rather than an API — "shade
  11:00-13:00 as lunch", "grey out weekends", "working hours are 9 to 5", "mark today", "draw
  arrows between tasks", "group by city", "a totals row", "a search box", "export to Excel",
  "print", "undo", "swimlanes", "baselines", "critical path", "recurring events", "split a task",
  "WIP limit", "highlight that row". Most of these are one config.
metadata:
  tags: bryntum, features, scheduler, schedulerpro, gantt, grid, calendar, taskboard, timeranges, nonworkingtime, export, undo, tier
---

# Bryntum features

Most outcome-style requests map to a single entry in the `features` config. The usual mistake
is not knowing the feature exists and hand-building it with an `eventRenderer`, a `cellCls`,
positioned `<div>`s or CSS. That can look right while missing virtualization, export, state, or
(for working hours) the scheduling engine entirely. Check the index first; the reference files
have snippets, tiers and per-product differences.


## How features behave

- The key is the class name with a lowercase first letter: `GroupSummary` → `groupSummary`,
  `CriticalPaths` → `criticalPaths`. `true` enables with defaults, an object configures, `false`
  turns off one that is on by default.
- On 8.x every feature belongs to a tier — Community, Business or Enterprise, cumulative. The
  tier in the Index and the reference files is the lowest tier that has it; unmarked entries are
  in every tier. A feature above the app's tier is skipped and a dialog names the tier it needs,
  so say the tier when you suggest one. On 7.x there are no tiers. Of the Scheduler features
  marked Enterprise here, `resourceNonWorkingTime`, `timeSpanHighlight`, `nestedEvents`,
  `eventSegments`, `taskEdit` and `versions` were Scheduler Pro's (`@bryntum/schedulerpro`); the
  others were in `@bryntum/scheduler` or `@bryntum/grid`.
- Configure features at construction. On 8.x add a feature later with
  `widget.addFeature({ summary : true })` (unknown names throw). On 7.x a feature added after
  construction never runs its paint-time init — include it in the config and flip
  `widget.features.x.disabled = false`.
- The React and Vue wrappers take each feature as a `<name>Feature` prop (`nonWorkingTimeFeature`,
  `eventTooltipFeature={{ … }}`), not a `features` object. Runtime access is still
  `instance.features.<name>`.
- Registered on by default is not the same as active: Gantt's `criticalPaths` ships
  `disabled : true`.
- On 8.x defaults can depend on the tier — see editing-and-export.md.
- The same concept can have different names per product: Scheduler `eventEdit` vs Gantt
  `taskEdit`; Scheduler `eventTooltip` vs Gantt `taskTooltip`; `template` in Scheduler and Gantt
  tooltips vs `renderer` in Calendar's.

## Index

Raw: `https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-features/references/<file>.md`

| The user asks for | Use | Tier | Reference |
|---|---|---|---|
| A shaded band across the timeline (lunch, maintenance window) | `timeRanges` + `project.timeRanges` | Business | [timeline](references/timeline.md) |
| A band for one resource only (PTO) | `resourceTimeRanges` | Business | [timeline](references/timeline.md) |
| A "now" line | `timeRanges : { showCurrentTimeLine : true }` | Business | [timeline](references/timeline.md) |
| Grey out weekends / off-hours | `nonWorkingTime`, `eventNonWorkingTime`, `resourceNonWorkingTime`, `taskNonWorkingTime` — pick by where the grey goes | Business; `eventNonWorkingTime` and `resourceNonWorkingTime` Enterprise | [timeline](references/timeline.md) |
| Working hours that affect scheduling | Project `calendars` | Scheduler Enterprise; Gantt Business | [timeline](references/timeline.md) |
| Temporarily highlight a span from code | `timeSpanHighlight` | Enterprise | [timeline](references/timeline.md) |
| Arrows between events / tasks | `dependencies` (+ `dependencyEdit`) | Community (`dependencyEdit` Business) | [timeline](references/timeline.md) |
| Events inside a parent bar; splitting an event | `nestedEvents`; `eventSegments` | Enterprise | [timeline](references/timeline.md) |
| Highlight a row, a column, stripes | record `cls`, column `cellCls`, `stripe` | Community | [grid](references/grid.md) |
| Show the event / task name beside or below the bar | `labels` | Business | [timeline](references/timeline.md) |
| Copy / paste rows, cells or events | `rowCopyPaste`, `cellCopyPaste`, `eventCopyPaste`; Gantt: `taskCopyPaste` (on by default) replaces `rowCopyPaste` | Business | [grid](references/grid.md) |
| Drag rows to reorder | `rowReorder` | Enterprise (Gantt: Business, on by default) | [grid](references/grid.md) |
| Freeze / pin a column | `pinColumns` | Business | [grid](references/grid.md) |
| Expand a row to show details | `rowExpander` | Enterprise (Gantt: Business) | [grid](references/grid.md) |
| Tree, or a tree built from flat rows | `TreeGrid` / `tree`; `treeGroup` | Community; `treeGroup` Business | [grid](references/grid.md) |
| Group rows, totals row, per-group totals | `group`, `summary`, `groupSummary` | Business; `summary` and `groupSummary` Enterprise | [grid](references/grid.md) |
| Filter UI, search box, type-to-find, multi-sort | `filter` / `filterBar`, `search`, `quickFind`, `store.sort(…, true)` | Community; `filter` Business | [grid](references/grid.md) |
| Event/task editor, tooltips | `eventEdit` / `taskEdit`, `eventTooltip` / `taskTooltip` (Gantt), `cellTooltip` | Community; `taskEdit` Enterprise in Scheduler, Business in Gantt; `cellTooltip` Business | [editing & export](references/editing-and-export.md) |
| Undo / redo | Project `stm : { autoRecord : true }` plus an `UndoRedo` widget or `stm.enable()` — both halves are needed; Gantt imports the widget from the Scheduler package | Business (TaskBoard widget Enterprise) | [editing & export](references/editing-and-export.md) |
| Excel, PDF, print, MS Project | `excelExporter`, `pdfExport`, `print`, `mspExport` | Enterprise | [editing & export](references/editing-and-export.md) |
| Critical path, baselines, progress line, rollups, deadlines | Gantt features | Business to Enterprise, per feature | [gantt](references/gantt.md) |
| Resource load / utilisation | `ResourceHistogram` / `ResourceUtilization` views | Enterprise | [gantt](references/gantt.md) |
| Calendar views, recurring events, hiding weekends | `modes`, `recurrenceRule`, `hideNonWorkingDays` | Community; recurrence Business; advanced modes Enterprise | [calendar & taskboard](references/calendar-taskboard.md) |
| Swimlanes, card content, WIP limits | `swimlaneField`, `bodyItems`, (no WIP config) | Swimlanes Business; items per type | [calendar & taskboard](references/calendar-taskboard.md) |
| Drag from a grid or sidebar onto the timeline | `bryntum-drag-and-drop` skill | — | — |

## Things that do not exist

These are the most commonly invented APIs in this area:

- TaskBoard WIP limits — no config; build it with `beforeTaskDrop` and say it is custom.
- `timeRangesStore` / `resourceTimeRangesStore` — the stores are singular: `timeRangeStore`,
  `resourceTimeRangeStore`.
- A `taskFilter` TaskBoard feature — it is `columnFilter` or `filterBar`.
- `Grid#refresh()` on 7.x — Grid gained `refresh()` in 8.0 like the other products; on 7.x use
  `refreshRows()` / `refreshColumn()`.
- Multi-field grouping in the grid UI — use `treeGroup`.
- A Grid `UndoRedo` widget.
- `eventTooltip` in Gantt — it is `taskTooltip`. On 8.x an unknown feature key throws
  `Invalid type name` (on 7.x it was silently ignored).
- Client-only PDF export — `pdfExport` needs the export server.
- `columnLines` driven by time ranges — it draws from ViewPreset ticks only.

## Related

- `bryntum-editor` — customizing the event/task editor.
- `bryntum-styling` — event bar content and `eventRenderer`.
- `bryntum-drag-and-drop` — dragging onto the timeline from outside.
- `bryntum-from-extjs` — Ext JS idioms that do not apply here.
