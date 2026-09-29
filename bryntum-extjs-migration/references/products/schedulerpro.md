# Scheduler Pro

Support level: **moderate** (no official Ext guide; follows the Scheduler shape plus a project-centric data layer).

Finished example (extjs-migration-agent repo): `schedulerpro-extjsmodern-vite` (the Scheduler template plus a `project`
hub with calendars, assignments and dependencies, percent bars, grouped resources,
`eventStore.isDateRangeAvailable()` overlap checks).

## What a Bryntum Scheduler Pro app looks like

- `new SchedulerPro({ appendTo, project, columns, features, viewPreset, startDate, endDate, eventRenderer, tbar })`
- `project` holds `resources`, `events`, `assignments`, `dependencies`, `calendars` and runs the scheduling engine
- events have `startDate` + `duration`; `endDate` is derived; dependencies cascade

## Rules

1. **Reuse Scheduler for everything UI-shaped** — wrapper removal, shell, configs, toolbars, popups, renderers,
   `resourceTimeRanges`, dependency editor/menus. If a Scheduler mapping isn't contradicted by Scheduler Pro evidence,
   it holds.
2. **`project` is core data.** When the source has dependencies, assignments, calendars or constraint/effort behavior,
   use a `project` rather than ad hoc stores, and don't mark the migration complete if that behavior depends on
   dropped data.
3. **Pro-only behavior must be verified.** Dependency editing/highlighting, resource time ranges, calendar-driven
   scheduling and constraint/effort calculations: only claim them migrated with a verified mapping or an explicit
   product example/doc page; otherwise record the assumption in the report.
4. The editor feature is `taskEdit` (not `eventEdit`).

## Limits

For deeply engine-driven apps: reuse Scheduler guidance where valid, preserve every data structure you can map, and
document every unverified Scheduler Pro assumption.
