# Timeline features (Scheduler, Scheduler Pro, Gantt)

## Time bands and non-working time

| The user asks for | Use | Snippet |
|---|---|---|
| A band across the whole timeline | `timeRanges` + `project.timeRanges` | `features : { timeRanges : true }`, `project : { timeRanges : [{ startDate, endDate, name : 'Lunch', cls : 'lunch' }] }` |
| A band for one resource | `resourceTimeRanges` + `project.resourceTimeRanges` | `project : { resourceTimeRanges : [{ resourceId : 'r1', startDate, endDate, name : 'PTO' }] }` |
| A "now" line | `timeRanges` | `features : { timeRanges : { showCurrentTimeLine : true } }` |
| Grey the time axis (weekends, off-hours) | `nonWorkingTime` | `features : { nonWorkingTime : true }` |
| Grey the non-working part inside an event bar | `eventNonWorkingTime` (Scheduler) | `features : { eventNonWorkingTime : true }` |
| Grey per resource row, each with its own calendar | `resourceNonWorkingTime` (Scheduler Pro) | `nonWorkingTime` only knows the project calendar |
| Gantt: grey inside task rows or bars, on top of the time axis | `taskNonWorkingTime` | `features : { taskNonWorkingTime : { mode : 'row' } }` (or `'bar'`, `'both'`). Gantt weekends on the axis are still `nonWorkingTime` |
| Vertical tick lines | `columnLines` (already on for Scheduler / Gantt) | Draws from ViewPreset tick levels only — it cannot draw a line from a time range |

A time range is a `TimeSpan`: `startDate` plus `endDate` or `duration` + `durationUnit`, and
optional `name`, `cls`, `iconCls`, `style`. `ResourceTimeRangeModel` adds `resourceId` and
`timeRangeColor`.

Store properties are singular — `project.timeRangeStore`, `project.resourceTimeRangeStore`. The
inline data configs are plural (`timeRanges`, `resourceTimeRanges`); `timeRangesData` /
`resourceTimeRangesData` are deprecated as of 6.3.0.

## Working hours (Scheduler Pro / Gantt)

Non-working time in Pro and Gantt comes from the project calendar, which is also what makes the
engine skip weekends when scheduling. Grey boxes drawn on top look the same but don't affect
scheduling.

```javascript
const project = new ProjectModel({
    calendars : [{
        id                       : 'business',
        unspecifiedTimeIsWorking : false,
        intervals                : [
            { recurrentStartDate : 'at 08:00',        recurrentEndDate : 'at 17:00',        isWorking : true  },
            { recurrentStartDate : 'on Sat at 00:00', recurrentEndDate : 'on Mon at 00:00', isWorking : false }
        ]
    }],
    calendar : 'business'   // project default; resources and events can override via their `calendar` field
});
```

Use `calendars`, not the deprecated `calendarsData`.

## Highlighting and selecting time

- `timeSpanHighlight` (Pro / Gantt): `scheduler.features.timeSpanHighlight.highlightTimeSpan({ startDate, endDate })`,
  cleared with `unhighlightTimeSpans()`. Better than adding and removing a time range record.
- `timeSelection`: lets the user drag-select a span in the header.

## Dependencies

`dependencies` is off by default in Scheduler and on in Scheduler Pro and Gantt. Add
`dependencyEdit` (off by default) to let the user edit them.

```javascript
features : { dependencies : true },
project  : { dependencies : [{ from : 1, to : 2, type : 2 }] }
```

Persist `from` / `to`; `fromEvent` / `toEvent` are resolved accessors. `type` is an integer:
`0` StartToStart, `1` StartToEnd, `2` EndToStart (default), `3` EndToEnd.

## Nesting and segments

- `nestedEvents` (Scheduler Pro, off by default): nesting is a `children` array on the event,
  not a `parentId`.
- `eventSegments` (Pro, on by default): split with `await eventRecord.splitToSegments(splitDate, 2, 'day')`;
  also `setSegments()`, `mergeSegments()`.
- The `split` feature is unrelated — it splits the view into side-by-side panes.

## Rendered class names (for tests)

v7 class names are kebab-case; v6 names such as `.b-sch-nonworkingtime` match nothing.
Useful selectors: `.b-sch-non-working-time`, `.b-sch-dependency`, `.b-gantt-task-wrap.b-critical`
(critical path), `.b-gantt-task-tooltip`.
