# Scheduler

Support level: **strong** (an official Ext → Bryntum Scheduler guide exists; its mappings are corrected and folded into
`../api-mapping.md` §3, `../data-migration.md`, `../patterns.md`).

Scheduler is a Panel, so `title`, `tools` and `tbar` work. The official guide's "Scheduler is not a Panel" is
outdated. Colors come from `eventColor` (an event or resource field) and `eventStyle`, not inline styles injected by
renderers.

## Rules

- Set `barMargin`, `rowHeight`, `viewPreset`, `startDate`/`endDate` explicitly — defaults differ from Ext Scheduler.
- Resource column with name + avatar: `type : 'resourceInfo'` with `resourceImages` or `resourceImagePath`.
- Multi-resource events: `assignments` store instead of `resourceId` on events.
- Date + time field pairs in forms: `datefield` with `partner : '<timeFieldRef>'` plus a `timefield`, laid out
  with `layout : { type : 'box', horizontal : true, wrap : true, align : 'end' }` and `flex : '1 0 45%'` on each.
- Drag-create opens `eventEdit` for the new record. The name field is empty, with the model's `name` default or
  "New event" as its placeholder, and Cancel removes the record. If the Ext app created events silently, either keep
  the editor (a visible behavior change for the report) or turn it off. Without an event editor, drag-create adds the
  record directly, still named "New event" (or the model's `name` default). `eventEdit : false` also removes
  double-click editing. `beforeEventAdd` returning `false` vetoes the add. (SRC) **At the Core tier `eventEdit : false` removes the editor. At the Enterprise tier it holds only with
  `taskEdit : false` as well:** the Enterprise default editor is `taskEdit`, and with `eventEdit : false` alone it still
  opens on double-click and on drag-create (and intercepts pointer events). A custom editor on that tier configures
  `eventEdit` **and** sets `taskEdit : false` (tested, both vanilla and React).
- Open the editor for a new record from a button: `scheduler.editEvent(new eventStore.modelClass({...}))`.
- Editor title: set it in `beforeEventEditShow` (checked on 8.0.0).
- Ext validators (`dndValidatorFn`, `resizeValidatorFn`, `createValidatorFn`) → the `validatorFn` of `eventDrag`,
  `eventResize`, `eventDragCreate`. The argument is one context object and the result `{ valid, message }`
  (`api-mapping.md` §3; the resize shape is tested, the other two follow it: test the rejection paths).
- An Ext `beforeeventdrop` veto → `beforeEventDropFinalize` with `context.valid = false`. There is no
  `beforeEventDrop` event (checked in 8.0.0).
- Drag from an external list/grid onto the timeline: the `bryntum-drag-and-drop` skill.
- Time lines, zones, a hidden-hours axis, `zoomLevels`, hour header formats: `../api-mapping.md` §3.
- Two-line event bars (an Ext `eventBodyTemplate` with a header and a footer) need `stickyEvents : false` (otherwise the text
  clips to about 70 px) **and** `.b-sch-event-content { display : block }` (the default is a flex row, so the two DomConfig
  divs sit side by side).
- Epoch-millisecond dates, stores that don't load from `readUrl`, and sorted stores: `../data-migration.md`.

## Both types supported

- **Type A** (`Sch.*`, PascalCase data): full config/column/data mapping from the references.
- **Type B** (Bryntum Scheduler in an Ext wrapper): remove the Ext shell, copy the Scheduler config and re-check it
  against 8.0 defaults. Custom Ext dialogs usually become the built-in `eventEdit` plus a `Popup` subclass.
