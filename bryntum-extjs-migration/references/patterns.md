# Migration patterns

Idiom-level rewrites. Tags as in `api-mapping.md`. Examples use vanilla JS; §10 covers framework targets.

## 1. Application shell

- `Ext.application` + `Viewport` + controllers → one entry module creating widgets with `appendTo` (vanilla), or the
  framework's app component rendering the Bryntum wrapper component. (G-G)
- Remove every Ext script and stylesheet: `ext-all.js`, `bootstrap.js`, `app.json`, `classic.json`/`modern.json`,
  Sencha Cmd output, theme packages. (G-S)
- Ext layouts (`border`, `vbox`, `hbox`, `fit`) → a Bryntum `Container` with `layout : 'hbox'`/`'vbox'` and `flex`
  on children (for pure layout wrappers), or `Panel` if the layout container has panel features (header, tools). In a framework app, use CSS flexbox instead. `split : true` → `{ type : 'splitter' }` between items. (G-G, SRC)
- Multiple views sharing data → share one store or `project` instance. (G-G)
- Controller logic listening to global events → methods on a custom widget (e.g. a `Toolbar` subclass), plain
  functions, or framework services/hooks.

## 2. Custom classes

```js
// Ext
Ext.define('App.view.AppToolbar', { extend : 'Ext.Toolbar', xtype : 'apptoolbar', ... });

// Bryntum
import { Toolbar } from '@bryntum/core-<tier>';

export default class AppToolbar extends Toolbar {
    static type  = 'apptoolbar';
    static $name = 'AppToolbar';
    static configurable = { items : { /* keyed items */ } };

    // Resolve the host widget lazily: it isn't guaranteed to be set during construction,
    // and a cached reference goes stale if the toolbar is moved
    get host() {
        return this.owner;   // or this.up('gantt') / this.up('scheduler') when the toolbar is nested deeper
    }
}

AppToolbar.initClass();   // registers the type so { type : 'apptoolbar' } works
```

- Ext `initComponent` → `construct(...args)` + `super.construct(...args)`, but only for real setup logic. Most
  `initComponent` bodies just set configs, which belong in `static configurable`. (G-G)
- Ext `config : {}` + `applyX`/`updateX` → `static configurable = {}` (apply/update hooks: UNV, check docs).
- Don't subclass when a config on the stock widget does the job.
- Widgets have `callOnFunctions : true`, so a method named `on<EventName>` (e.g. `onBeforeEventDropFinalize`) is already
  a listener. Registering it again with `on()` runs it twice. Before naming a helper `on<X>`, check that the base class
  doesn't already fire an `x` event. (SRC)

## 3. Toolbar items and handlers (G-G, SRC)

```js
tbar : {
    items : {
        addButton   : { icon : 'fa fa-plus', text : 'Create', onClick : 'up.onAddClick' },
        filler      : '->',
        zoomButtons : { type : 'buttonGroup', items : { zoomIn : { icon : 'fa fa-search-plus', onClick : 'up.onZoomInClick' } } },
        undoRedo    : { type : 'undoredo', items : { transactionsCombo : null } }
    }
}
```

- `'up.name'` calls `name(event)` on the nearest ancestor that has a `name` property. Point it only at methods written
  as handlers. Two traps (SRC):
  - `'up.hide'` on a Popup's bbar button finds the bbar `Toolbar` first. The toolbar disappears and the modal stays
    open. Use `'up.onCancelClick'` with `onCancelClick() { this.hide(); }`.
  - API methods that take arguments (`shiftNext(amount)`, `zoomIn(levels)`, `hide(animate)`) receive the click event
    as their argument. Wrap them: `onNextClick() { this.shiftNext(); }`.
- Access items by key: `this.widgetMap.addButton`. Ext `reference` / `itemId` → the key in `items`.
- Ext `header : { items }` → Panel `tools` (Grid, Scheduler and Gantt are Panels).

## 4. ViewModel bindings → explicit state

An Ext `bind : '{rowHeight}'` shared by two components becomes an explicit value on each, kept in sync by an
`onChange : 'up.onRowHeightChange'` handler on the field. In a framework target, hold the value in framework state and
pass it to the wrapper as a prop. ViewModel `formulas` become computed values: a calculated model field, a getter, or
framework computed state. (SRC)

## 5. Event / task editor

Keep the built-in editor (see the `bryntum-editor` skill). Ext apps often replaced the editor with a custom Ext dialog
(`beforeeventedit` → `return false` + `Ext.Dialog`). Compare the dialog's fields with the built-in editor first —
they often match — and customize the built-in one via keyed `items` + `weight` (built-ins at 100, 200, ...; `null`
removes one). (SRC)

```js
features : {
    eventEdit : {
        items : {
            resourceField  : { label : 'Assigned' },            // only the properties you change
            startTimeField : { step : '30min' },
            endTimeField   : { step : '30min' },
            locationField  : { type : 'text', name : 'location', label : 'Location', weight : 120 }
        }
    }
},
listeners : {
    beforeEventEditShow({ editor, eventRecord }) {
        editor.title = eventRecord.isCreating ? 'New event' : 'Edit event';   // title set here, not earlier
    }
}
```

Gantt: `taskEdit : { items : { generalTab : { items : { customField : { type : 'textfield', name : 'customField', label : 'Custom', weight : 150 } } } } }` (G-G).

Show/hide fields per value: tag fields with `dataset`, toggle `widget.hidden` in a `change` listener. (G-S)

Only build a custom dialog when the user explicitly asks — then keep the feature enabled and return `false` from
`beforeEventEdit`/`beforeTaskEdit` (see `bryntum-editor`).

## 6. Other dialogs → `Popup` subclass (vanilla) (SRC)

```js
import { Popup } from '@bryntum/core-<tier>';

export default class RangeEditor extends Popup {
    static $name = 'RangeEditor';
    static type  = 'rangeeditor';

    static configurable = {
        title     : 'Time Range',
        centered  : true,
        modal     : true,
        closable  : true,
        autoShow  : false,
        autoClose : false,
        width     : '32em',
        layout    : { type : 'box', horizontal : true, wrap : true, align : 'end' },
        defaults  : { required : true, clearable : false },
        items     : {
            nameField      : { type : 'textfield', label : 'Name', flex : '1 0 100%' },
            startDateField : { type : 'datefield', label : 'Start', partner : 'startTimeField', flex : '1 0 45%' },
            startTimeField : { type : 'timefield', ariaLabel : 'Start time', step : '30min', flex : '1 0 45%' }
        },
        bbar : {
            items : {
                saveButton   : { text : 'Save', rendition : 'filled', onClick : 'up.onSaveClick' },
                cancelButton : { text : 'Cancel', onClick : 'up.onCancelClick' }   // not 'up.hide' (§3)
            }
        },
        keyMap : { Enter : 'onSaveClick' }   // Ext dialogs often saved on Enter
    };

    onSaveClick() {
        const { nameField, startDateField, startTimeField } = this.widgetMap;
        if ([nameField, startDateField, startTimeField].every(field => field.isValid)) {
            // write back to the store, then:
            this.hide();
        }
    }

    onCancelClick() {
        this.hide();
    }
}

RangeEditor.initClass();
```

Checked against 7.3.7 (recheck on 8.x): Cancel closes the dialog, and Enter saves only when the fields are valid.
In a framework app that already has a component system (MUI, Angular Material, Vuetify, ...), use that system's dialog
for app-level dialogs instead (base skill, widget-first rule).

## 7. Rendering

- Scheduler: `eventRenderer({ eventRecord, resourceRecord, renderData })` — set `renderData.cls` / `renderData.style`,
  and return a DomConfig (preferred) or an HTML string. In an HTML string, escape every record value with the
  `StringHelper.xss` tagged template or `StringHelper.encodeHtml()`: Ext `XTemplate`s often encoded by default, so a
  direct port can open an XSS hole. Custom layouts: see the `bryntum-styling` skill. (SRC)
- Gantt: `taskRenderer({ taskRecord, renderData })`. (SRC)
- Colors painted in renderers (`background-color : resource.color`) → `eventColor` on the event or resource (any CSS
  color) plus `eventStyle`. (SRC)
- Row CSS class (`getRowClass`) → `cls` field on the record, or a column `renderer`. (G-G)
- Grid group headers (Ext `groupHeaderTpl`): `features : { group : { field, renderer({ groupRowFor, count, isFirstColumn }) { return isFirstColumn ? '…' : ''; } } }`.
  Per column: `column.groupRenderer({ groupRowFor, count, groupColumn })`, which gets no `isFirstColumn`. Don't port
  checks like `data.column === data.grid.columns.first`. (SRC)
- Tooltips: the product's tooltip feature (`eventTooltip`, `taskTooltip`, `cellTooltip`) with a `template`/renderer
  function. `cellTooltip` is off by default in Grid: a column `tooltipRenderer` does nothing until you set
  `features : { cellTooltip : true }`. (SRC / DOC per product)

## 8. Localization

```js
import { LocaleHelper } from '@bryntum/core-<tier>';
import '@bryntum/<product>-<tier>/locales/<product>.locale.Es'; // 7.x: '@bryntum/<product>/locales/...'

export default LocaleHelper.publishLocale({
    localeName : 'Es', localeDesc : 'Español', localeCode : 'es',
    Button : { Create : 'Crear' }          // app strings grouped by class name
});
```

- Ext locale overrides → one file per locale as above.
- Strings in configs become `'L{Text}'`; in code `this.L('L{Text}')`.
- Switch: `localeManager.applyLocale(name)` (available on widgets as `this.localeManager`). A language combo is built from `localeManager.locales`, not a
  hand-written list (see `products/gantt.md`).

## 9. Undo / redo

Ext state-tracking manager → the project/store `stm`: `project : { stm : { autoRecord : true } }` plus an `undoredo`
toolbar widget (Gantt, Scheduler Enterprise — G-G). For Scheduler/Grid, confirm the StateTrackingManager wiring in the docs.

## 10. Framework targets

The steps are the same for every target. Only the place where the config lives changes, and the framework skill covers
the wrapper.

| Concern | Vanilla | React | Angular | Vue 3 |
|---|---|---|---|---|
| Instance access | variable / `window.gantt` | `ref.current.instance` | `@ViewChild(...).instance` | template ref `.instance` |
| Custom Bryntum classes (`Toolbar`/`Popup` subclasses) | `lib/*.js` | same classes, imported and referenced by `type` | same | same |
| ViewModel binds | handlers | React state → props | component state/inputs → bindings | refs/reactive → props |
| App dialogs outside Bryntum | `Popup` | app's component system if any, else `Popup` | same | same |

Keep Bryntum configuration inside Bryntum configs even in a framework app: toolbars, editors, menus and renderers stay
Bryntum widgets, not framework components.

## 11. Incremental path: Bryntum inside the Ext shell

Only when the team accepted an interim state (`SKILL.md`, "Before starting"). Replace the Ext/Sch/Gnt component with a thin
`Ext.Component` that owns one Bryntum widget, and keep the Bryntum config modern (no legacy translation layer):

```js
Ext.define('Bryntum.wrapper.Scheduler', {
    extend : 'Ext.Component',
    alias  : 'widget.bryntum-scheduler',
    config : { bryntumConfig : null },

    afterRender() {
        this.callParent(arguments);
        this.bryntumWidget = new bryntum.scheduler.Scheduler(
            Ext.apply({ appendTo : this.el.dom }, this.getBryntumConfig())
        );
    },
    onDestroy() {
        this.bryntumWidget?.destroy();
        this.bryntumWidget = null;
        this.callParent(arguments);
    }
});
```

- **Loading Bryntum next to Ext (8.0).** There is no prebuilt `*.umd.js`. Either load the package as an ES module and
  publish what Ext code needs on `window` (the suite's `extjsmodern` demos do `window.bryntum.scheduler = scheduler`), or
  build a UMD bundle yourself (the product's `gettingstarted/build-umd.md`: webpack, publishes the `bryntum.<product>`
  global). Class access such as `bryntum.scheduler.Scheduler` only works with the second route or your own global.
  7.x ships `build/<product>.umd.js` (`v7.md`).
- **CSS coexistence.** Load the Bryntum sheets (structural + one theme + Font Awesome, base skill "CSS setup") next to the
  Ext theme. Bryntum rules are scoped under `.b-widget`. If layout breaks, wrap the widget in `position : relative;
  overflow : hidden`. Tooltip/popup stacking: adjust `z-index` rather than patching either library.
- **Resizing.** Bryntum observes its own size; if the Ext layout doesn't give the wrapper a height, the widget falls back
  to `minHeight`. Give the Ext item `flex` or `height`.
- **Stores.** Prefer separate stores: the Bryntum project loads from the same endpoints the Ext stores use. A one-way copy
  from an Ext store (`store.getRange().map(r => r.getData())`) into the project is acceptable for read-only data; two-way
  sync is not worth it. Count records on both sides.
- **License key.** The Core and Enterprise tiers need `bryntum.licenseKey` (or the trial runs). Set it before the wrapper creates the widget.

## 12. Custom view presets (Ext Scheduler / Gantt)

Built-in preset names (`weekAndDay`, `dayAndWeek`, ...) carry over. Custom presets registered with
`Sch.preset.Manager.registerPreset` change shape; register with `PresetManager` from the Scheduler package
(`@bryntum/scheduler-<tier>`; the Gantt package doesn't re-export it, so a Gantt app imports it from the Scheduler package too and declares that dependency). Names checked against the 8.0.0 `ViewPreset` source:

| Ext preset config | Bryntum preset config |
|---|---|
| `timeColumnWidth` | `tickWidth` |
| `headerConfig : { top, middle, bottom }` | `headers : [...]` ordered top to bottom |
| `headerConfig.<row>.dateFormat : 'd M Y'` | `headers[n].dateFormat : 'DD MMM YYYY'` (convert tokens, `api-mapping.md` §7) |
| `headerConfig.middle.headerCls` | `headers[n].headerCellCls` |
| `displayDateFormat : 'G:i'` | `displayDateFormat : 'H:mm'` |
| `shiftUnit : 'DAY'`, `timeResolution : { unit : 'MINUTE' }` | lowercase units: `'day'`, `'minute'` |
| `renderer(start, end, cfg)` in a header row | `renderer(start, end, cfg)` in `headers[n]` (check the arguments in the docs) |

Where several screens share one preset list, port it once into a module and import it.

