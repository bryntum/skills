---
name: bryntum-extjs-wrapper
description: >
  Write an Ext JS component (Modern or Classic toolkit) that wraps any Bryntum widget (Grid, Scheduler, Gantt,
  Calendar, TaskBoard, or any Widget subclass) so an Ext JS app uses it like a native Ext component: `xtype`, Ext
  configs, `getXxx()`/`setXxx()`, ViewModel `bind`, Ext `listeners` and ViewController methods. The generated class
  is self-contained and introspects the Bryntum class instead of hardcoding a property list. Use when embedding
  Bryntum inside an Ext JS application, on "Bryntum in Ext JS", "Ext JS wrapper for Scheduler/Gantt/Calendar",
  "xtype for Bryntum", "Ext.define a Bryntum panel", or when a hand-written Ext wrapper (`exportedProperties`,
  `createPropertyProxy`) needs to expose more configs. Use alongside the `bryntum` skill.
metadata:
  tags: bryntum, extjs, sencha, wrapper, integration, scheduler, gantt, grid, calendar, taskboard
---

# Bryntum widget inside an Ext JS app

Keeps the Ext JS app and embeds a Bryntum widget as an Ext component. To remove Ext JS altogether use
`bryntum-extjs-migration` instead.

Write **one self-contained ES module per wrapped Bryntum class** (for example `Bryntum/CalendarPanel.js`) that
imports the Bryntum class and calls `Ext.define`. Do not create or import a shared wrapper library, and never write a
hardcoded list of exported configs: the configs are discovered from the Bryntum class at load time.

- `references/example-calendar-panel.md`: a complete generated wrapper (Modern `Ext.Panel` around a Calendar). Use it
  as the template and adapt it with the steps below.
- `references/wrapper-rules.md`: the name lists (wrapper-owned names, Ext instance fields, Ext event names, Ext JS
  aliases) and the Classic toolkit differences.

## Recipe

1. **Module scope.** Import the Bryntum class from the product's module bundle, `build/<product>.module.js` in the
   product distribution (adjust the relative path), or from the npm package (`@bryntum/calendar-<tier>` on 8.0). Keep it in a module constant (`WidgetClass = Calendar`). Never put
   it, or any other class or non-method function, in the `Ext.define` body: Ext stamps `$name`/`$owner` on body
   functions, which renames the Bryntum class itself and breaks its CSS classes (`b-widget-class` for `b-calendar`).
   Pick the Ext base: `Ext.Panel` keeps an Ext header for titles and tools; `Ext.Component` gives the whole element.
   A Classic `Ext.panel.Panel` needs `layout : 'fit'` in the class body: its default `auto` layout renders an
   `x-autocontainer-outerCt` into the body, which `adopt` keeps and which then takes the widget's height.
2. **Introspect** (once, at module load):
   - every name in `WidgetClass.$meta.configs` (`for...in`, it chains to the superclass configs, so it covers the
     whole hierarchy, `static configurable` and `defaultConfig`, mixins included);
   - every prototype accessor with a setter, walking the prototype chain (plain `set foo()` properties);
   - drop names not starting with a lowercase letter (`_x`, `$x`, `X`), `is*`/`has*` flags, and the wrapper-owned
     names (`adopt`, `appendTo`, `renderTo`, `element`, `id`, ...).
3. **Ext wins clashes.** A name is not exposed when the Ext base owns it: a config of the Ext class
   (`getConfigurator().configs`), a prototype member, an existing `getXxx`/`setXxx`, or an Ext instance field (`el`,
   `body`, `items`, `layout`, ...). Collect them as `bryntumClashes` and the rest as `bryntumProperties`. The Bryntum
   value of a clashing name goes in an instance `widgetConfig` object (`widgetConfig : { tbar : {...} }`).
4. **Ext JS names.** Optionally accept the Ext JS names Ext developers reach for and map them to the Bryntum name
   (`eventEditor` → `features.eventEdit` for a Calendar). Only add aliases that fit the product, and only when the
   Bryntum class has no such name and Ext does not own it. Feature aliases go in `featureAliases` (as in the
   example), plain renames in a `configAliases` map, shape-changing ones need their own setter; see
   `references/wrapper-rules.md`. Products with no matching Ext JS names (Grid, TaskBoard) keep both maps empty.
   The full mapping is in the `bryntum-extjs-migration` skill (`references/api-mapping.md`).
5. **Constructor.** Split the config: exposed names and aliases go to a pending Bryntum config, in the order
   class-body defaults (`me[name]`, so Ext subclasses can declare defaults), `widgetConfig`, instance configs; the
   rest goes to Ext. Call the Ext constructor as `extProto.constructor.call(me, extCfg)`: `callParent()` needs
   `Function.caller`, which ES module code does not have.
6. **Accessors.** For each exposed name add `getXxx()`/`setXxx()` to the body. Before the widget exists they read and
   write the pending config, afterwards the widget. The setter is what ViewModel `bind` calls.
7. **Create once sized.** Modern: in `initialize`, a single `painted` listener (an arrow function: a string handler
   would resolve to the ViewController). Classic: `onBoxReady` (call `extProto.onBoxReady` first); `afterRender`
   runs before Classic's first layout, when the body has no size yet. Create the widget with `adopt` on the Ext element:
   `bodyElement` (Modern Panel) / `body` (Classic Panel), else `el` / `element`. Then:
   - before construction, save the element's inline styles, its computed `flex`, and its classes **without** Ext's
     `x-` classes (pass those into Bryntum's `cls`). `adopt` clears classes and inline styles; keeping `x-` classes
     would let Ext theme rules hit the widget;
   - after construction, put Ext's inline layout styles back, each only where Bryntum set none. Classic sizes the
     body with inline `width`/`height`, so that is all it needs. Modern sizes it with
     `.x-container-body-el { flex : 1 }`, which goes with the `x-` classes: also set the saved `flex` inline, or the
     widget shrinks to its content (the Calendar hides this with its own `height : 100%`). Other class-based layout
     styles (`position`, `overflow`, `display`) match Bryntum's and need nothing;
   - set an inline `font-size : initial` unless Ext set one: Bryntum sizes in `em`, and Ext themes set small font
     sizes on the panel bodies around the widget (Material 12px), so it would render tiny;
   - add no sizing CSS, wrapper `<div>` or resize syncing: the widget *is* the Ext element, Ext's layout sizes it and
     Bryntum re-lays itself out from its own resize monitoring;
   - fire `widgetcreate(cmp, widget)`, the place to wire stores and listeners.
8. **Events.** Relay all Bryntum events with a `catchAll` listener as Ext events: lower-cased, `(cmp, event)`, only
   when `hasListeners[name]`, returning the Ext result so `false` vetoes `before*` events. Take the name from
   `event.eventName`, not `event.type`: Bryntum re-triggers some objects (`drag` in `beforeDropExternal` →
   `dropExternal`) and `type` keeps the first name. Prefix names Ext uses for the component lifecycle with `widget`
   (`resize` → `widgetresize`), using the full Modern + Classic list in `references/wrapper-rules.md`.
9. **ViewController callbacks.** Before creating the widget, and in setters, replace string values of keys ending in
   `Renderer`, `renderer`, `Fn`, `Template`, `Tpl`, `Handler`, `handler` that are plain identifiers with
   `(...args) => Ext.callback(name, null, args, 0, cmp)`, walking nested objects and arrays (`modeDefaults`,
   `columns`) but not data keys (`*data`, `*Store`, `project`, `crudManager`). Leave `'up.x'` strings to Bryntum.
10. **Destroy.** Override `destroy`: destroy the Bryntum widget first (its DOM still exists, and Bryntum restores the
    element's original classes and styles), then call `extProto.destroy`.
11. **Ext compat, once.** Delete `transitionend`, `animationstart` and `animationend` from
    `Ext.dom.Element.prototype.eventMap`: Ext maps them to `webkit*` names Chrome no longer fires, and Bryntum
    animations would hang.
12. **A getter for the widget** (`getCalendar()`, returning `null` before creation and after destroy), and expose
    `bryntumProperties`/`bryntumClashes` on the prototype so app code can check names instead of guessing.

## Using the generated component

Load Ext JS with a classic `<script>` (`ext-modern-all-debug.js` for Modern, `ext-all-debug.js` for Classic) and its
theme CSS, then the Bryntum CSS, and the app as `<script type="module">`. The entry module imports the wrapper module
before `Ext.application` runs, so `requires : ['Bryntum.CalendarPanel']` does not make `Ext.Loader` fetch anything.
Skeletons with URLs: Modern in `references/example-calendar-panel.md`, Classic (Viewport, border layout, `tbar`
checkbox bound to a Bryntum config) in `references/wrapper-rules.md`. A Modern item:

```javascript
{
    xtype        : 'calendarpanel',
    reference    : 'calendarPanel',
    title        : 'Team calendar',                 // Ext Panel config (clash)
    flex         : 1,
    bind         : { mode : '{mode}' },             // calls the generated setMode()
    date         : new Date(2026, 10, 25),          // Bryntum config
    project      : { loadUrl : 'data/data.json', autoLoad : true },
    modeDefaults : { eventRenderer : 'renderEvent' },   // ViewController method
    widgetConfig : { tbar : { items : { modeSelector : false } } }, // Bryntum value of a clashing name
    listeners    : {
        widgetcreate     : 'onCalendarCreate',      // (cmp, calendar)
        activeitemchange : 'onModeChange',          // relayed Bryntum event: (cmp, event)
        eventclick       : 'onEventClick'
    }
}
```

## Gotchas

- **No two-way binding.** Ext `publishes`/`twoWayBindable` need Ext configs. Push changes back from a relayed event,
  e.g. `activeitemchange : cmp => vm.set('mode', cmp.getMode())`.
- **The widget exists only after first paint.** Wire stores and listeners in `widgetcreate`, not in
  `ViewController.init`.
- **CSS.** Load the product CSS and a Bryntum theme (`calendar.css` + `visby.css`, or the package equivalents) next to
  the Ext theme; they do not collide (`x-` vs `b-`). Also load `build/fontawesome/css/fontawesome.css` and
  `solid.css`: 8.0 ships Font Awesome separately, without it Bryntum icons are empty boxes. Ext's `x-fa` (Font
  Awesome 5) and Bryntum's (Font Awesome 6) do not collide. A `material3` theme is closest to Ext Material. With
  `color-scheme : light dark` on `<html>`, Bryntum follows the OS dark mode while most Ext themes stay light: set
  `color-scheme : light` for a light-only Ext theme.
- **Font size.** To size the widget, pass `widgetConfig : { style : { fontSize : '14px' } }` (`style` is Ext's
  otherwise); app CSS would need `!important` against the inline `font-size : initial`.
- **Calendar.** A top-level `eventRenderer` is ignored: use `modeDefaults : { eventRenderer }`. Its `tbar` clashes
  with `Ext.Panel`'s. When an Ext field picks the mode, hide the Calendar's selector with
  `widgetConfig : { tbar : { items : { modeSelector : false } } }`, but keep the `activeitemchange` sync: clicking a
  date header still drills into day view. External drag sources: `features.externalEventSource` with
  `dragRootElement`, `dragItemSelector` and `getRecordFromElement` (Core tier), and the relayed `dropexternal` event.
- **Grid.** Inline `data` clashes with Ext's template `data`: use `widgetConfig : { data }` or `store`.
- **TaskBoard.** Pass `columnField`, `columns` and `project : { tasks }` directly. `tbar`, `bbar`, `keyMap`,
  `collapsible` and `record` clash on `Ext.Panel`: use `widgetConfig`. `taskClick` relays as
  `taskclick(cmp, { taskRecord, columnRecord })`. No Ext JS aliases.
- **Tier.** On 8.0 set `tier` like any Bryntum config (class body or instance) only to run a package at a lower tier
  than its own; leave it out otherwise.
- **Destroy order.** Destroy the Ext component, never the Bryntum widget alone.
