# Scheduler Enterprise (formerly Scheduler Pro)

On 8.0 Scheduler Pro is no longer a product: it is `Scheduler` at the **Enterprise** tier (`@bryntum/scheduler-enterprise`,
class `Scheduler`, wrapper `BryntumScheduler`). The instance `tier` config defaults to the package tier. On 7.x it is
`SchedulerPro` from `@bryntum/schedulerpro` (see `../v7.md`). The file name is kept for existing links.

Support level: **moderate** (no official Ext guide; follows the Scheduler shape plus a project-centric data layer).

`project` holds `resources`, `events`, `assignments`, `dependencies` and `calendars`, and runs the scheduling engine.
Events have `startDate` + `duration`. `endDate` is derived, and dependencies cascade.

## Rules

- For anything UI-shaped, reuse the Scheduler mapping: wrapper removal, shell, configs, toolbars, popups,
  renderers, `resourceTimeRanges`, dependency editor/menus. A Scheduler mapping holds unless Scheduler Enterprise evidence
  contradicts it.
- `project` is core data. When the source has dependencies, assignments, calendars or constraint/effort behavior, use a
  `project` rather than ad hoc stores. The migration isn't complete while that behavior depends on dropped data.
- Enterprise-engine behavior needs evidence: dependency editing/highlighting, resource time ranges, calendar-driven scheduling,
  and constraint/effort calculations. Claim one as migrated only with a verified mapping or a product example/doc page.
  Otherwise record the assumption in the report.
- The editor feature is `taskEdit`, not `eventEdit` (Enterprise default; at the Community/Core tier it is `eventEdit`).
- A Scheduler app at a lower tier has no engine: use the plain Scheduler mapping and report any dependency/calendar
  behavior the Ext app had as a gap.

## Limits

For deeply engine-driven apps: reuse Scheduler guidance where valid, preserve every data structure you can map, and
document every unverified Scheduler Enterprise assumption.
