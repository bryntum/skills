# Gantt and Scheduler Enterprise features

Tier marks are 8.x; an unmarked feature is in every tier. Everything below needs Gantt Business or
Enterprise (Community runs the lighter engine — see timeline.md).

| The user asks for | Use | Tier | Snippet / note |
|---|---|---|---|
| Critical path | `criticalPaths` — registered on but starts disabled | Enterprise | `gantt.features.criticalPaths.disabled = false` |
| Baselines / planned vs actual | `baselines` (off) + task `baselines` field | Enterprise | `{ baselines : [{ startDate : '2026-01-01', endDate : '2026-01-05' }] }` |
| Capture current dates as a baseline | `setBaseline(version)` — 1-based | Enterprise | `gantt.taskStore.setBaseline(1)` |
| A progress / status line | `progressLine` (off; needs `percentBar`, on for Gantt) | Business | `features : { progressLine : { statusDate : new Date() } }` |
| Children summarized on the parent bar | `rollups` (off) + `rollup : true` on tasks | Enterprise | |
| Deadline / constraint markers | `indicators` (off) — `earlyDates`, `lateDates`, `constraintDate`, `deadlineDate` | Business | `indicators : { items : { deadlineDate : true, earlyDates : false } }` |
| Percent-done bar | `percentBar` (on) + `percentDone` field | Business | |
| Project start / end lines | `projectLines` (on) | Business | `projectLines : { showStatusDate : true }` |
| Shade the span of a parent's children | `parentArea` (off) | Business | |
| Backward (ALAP) scheduling | `project.direction` | Business (Scheduler: Enterprise) | `project : { direction : 'Backward' }` |
| Save / compare versions | `versions` (off, Gantt + Scheduler) | Enterprise | |

## Resource load (Scheduler Enterprise)

`ResourceHistogram` (load per resource) and `ResourceUtilization` (assignments per resource per
period) are view classes, not features, and Enterprise only. Import them from
`@bryntum/scheduler-<tier>` — the Gantt package does not export them. Give them the same `project`
as the scheduler, or link them with `partner` so they scroll together.
