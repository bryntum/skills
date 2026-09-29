# Scheduler

Support level: **strong** (an official Ext → Bryntum Scheduler guide exists; its mappings are corrected and folded into
`../api-mapping.md` §3, `../data-migration.md`, `../patterns.md`).

Finished examples (extjs-migration-agent repo): `scheduler-extjsmodern-vite` (the template: custom Ext dialogs →
built-in `eventEdit` + a `Popup` subclass, renderer colors → `eventColor`, calculated-field grouping, `Toast`),
`scheduler-extjs-multiassign-vite` (minimal wrapper removal, `crudManager.loadUrl` with multi-resource `assignments` +
`dependencies`, `resourceInfo` avatars, `eventStyle : 'bordered'`).

## What a Bryntum Scheduler app looks like

- `new Scheduler({ appendTo, columns, resourceStore, eventStore | crudManager, features, viewPreset, startDate, endDate })`
- a Panel in 7.x: `title`, `tools`, `tbar` work (the official guide's "Scheduler is not a Panel" is outdated)
- behavior through `features` (`dependencies`, `eventEdit`, `eventTooltip`, `eventMenu`, `stripe`, `timeRanges`, ...)
- colors through `eventColor` (event or resource field) and `eventStyle`, not renderer-injected inline styles

## Rules

- Set `barMargin`, `rowHeight`, `viewPreset`, `startDate`/`endDate` explicitly — defaults differ from Ext Scheduler.
- Resource column with name + avatar: `type : 'resourceInfo'` with `resourceImages` or `resourceImagePath`.
- Multi-resource events: `assignments` store instead of `resourceId` on events.
- **Date + time field pairs** in forms: `datefield` with `partner : '<timeFieldRef>'` plus a `timefield`, laid out
  with `layout : { type : 'box', horizontal : true, wrap : true, align : 'end' }` and `flex : '1 0 45%'` on each.
  Don't use a combined `datetimefield` with the Material3 theme — its label overlaps the input.
- Open the editor for a new record from a button: `scheduler.editEvent(new eventStore.modelClass({...}))`.
- Editor title: set it in `beforeEventEditShow` (earlier hooks are overwritten in 7.3.7).
- Ext validators (`dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn`) → the `validatorFn` of `eventDrag`,
  `eventResize`, `eventDragCreate` (UNV — verify the argument shape and test the rejection paths).
- Drag from an external list/grid onto the timeline: the `bryntum-drag-and-drop` skill.

## Both types supported

- **Type A** (`Sch.*`, PascalCase data): full config/column/data mapping from the references.
- **Type B** (Bryntum Scheduler in an Ext wrapper): use `scheduler-extjsmodern-vite` as the template.

No finished type A example exists yet.
