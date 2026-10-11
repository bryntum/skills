# Gantt and Scheduler Pro features

| The user asks for | Use | Snippet / note |
|---|---|---|
| Critical path | `criticalPaths` — registered on but starts disabled | `gantt.features.criticalPaths.disabled = false` |
| Baselines / planned vs actual | `baselines` (off) + task `baselines` field | `{ baselines : [{ startDate : '2026-01-01', endDate : '2026-01-05' }] }` |
| Capture current dates as a baseline | `setBaseline(version)` — 1-based | `gantt.taskStore.setBaseline(1)` |
| A progress / status line | `progressLine` (off; needs `percentBar`, on for Gantt) | `features : { progressLine : { statusDate : new Date() } }` |
| Children summarized on the parent bar | `rollups` (off) + `rollup : true` on tasks | |
| Deadline / constraint markers | `indicators` (off) — `earlyDates`, `lateDates`, `constraintDate`, `deadlineDate` | `indicators : { items : { deadlineDate : true, earlyDates : false } }` |
| Percent-done bar | `percentBar` (on) + `percentDone` field | |
| Project start / end lines | `projectLines` (on) | `projectLines : { showStatusDate : true }` |
| Shade the span of a parent's children | `parentArea` (off) | |
| Backward (ALAP) scheduling | `project.direction` | `project : { direction : 'Backward' }` |
| Save / compare versions | `versions` (off, Gantt + Scheduler Pro) | |

## Resource load (Scheduler Pro)

`ResourceHistogram` (load per resource) and `ResourceUtilization` (assignments per resource per
period) are view classes, not features. Give them the same `project` as the scheduler, or link
them with `partner` so they scroll together.
