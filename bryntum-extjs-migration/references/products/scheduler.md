# Scheduler

Support level: **strong** (an official Ext → Bryntum Scheduler guide exists; its mappings are corrected and folded into
`../api-mapping.md` §3, `../data-migration.md`, `../patterns.md`).

Scheduler is a Panel in 7.x, so `title`, `tools` and `tbar` work. The official guide's "Scheduler is not a Panel" is
outdated. Colors come from `eventColor` (an event or resource field) and `eventStyle`, not inline styles injected by
renderers.

## Rules

- Set `barMargin`, `rowHeight`, `viewPreset`, `startDate`/`endDate` explicitly — defaults differ from Ext Scheduler.
- Resource column with name + avatar: `type : 'resourceInfo'` with `resourceImages` or `resourceImagePath`.
- Multi-resource events: `assignments` store instead of `resourceId` on events.
- Date + time field pairs in forms: `datefield` with `partner : '<timeFieldRef>'` plus a `timefield`, laid out
  with `layout : { type : 'box', horizontal : true, wrap : true, align : 'end' }` and `flex : '1 0 45%'` on each.
  (7.3.7 bug: a combined `datetimefield` label overlaps the input in Material3.)
- Drag-create opens `eventEdit` for the new record. The name field is empty, with the model's `name` default or
  "New event" as its placeholder, and Cancel removes the record. If the Ext app created events silently, either keep
  the editor (a visible behavior change for the report) or turn it off. Without an event editor, drag-create adds the
  record directly, still named "New event" (or the model's `name` default). `eventEdit : false` also removes
  double-click editing. `beforeEventAdd` returning `false` vetoes the add. (SRC)
- Open the editor for a new record from a button: `scheduler.editEvent(new eventStore.modelClass({...}))`.
- Editor title: set it in `beforeEventEditShow` (earlier hooks are overwritten in 7.3.7).
- Ext validators (`dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn`) → the `validatorFn` of `eventDrag`,
  `eventResize`, `eventDragCreate` (UNV — verify the argument shape and test the rejection paths).
- An Ext `beforeeventdrop` veto → `beforeEventDropFinalize` with `context.valid = false`. There is no
  `beforeEventDrop` event in 7.x.
- Drag from an external list/grid onto the timeline: the `bryntum-drag-and-drop` skill.

## Both types supported

- **Type A** (`Sch.*`, PascalCase data): full config/column/data mapping from the references.
- **Type B** (Bryntum Scheduler in an Ext wrapper): remove the Ext shell, copy the Scheduler config and re-check it
  against 7.x defaults. Custom Ext dialogs usually become the built-in `eventEdit` plus a `Popup` subclass.
