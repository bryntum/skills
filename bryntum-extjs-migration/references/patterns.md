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
import { Toolbar } from '@bryntum/<product>';

export default class AppToolbar extends Toolbar {
    static type  = 'apptoolbar';
    static $name = 'AppToolbar';
    static configurable = { items : { /* keyed items */ } };

    construct(...args) {
        super.construct(...args);
        this.host = this.parent;
    }
}

AppToolbar.initClass();   // registers the type so { type : 'apptoolbar' } works
```

- Ext `initComponent` → `construct(...args)` + `super.construct(...args)`. (G-G)
- Ext `config : {}` + `applyX`/`updateX` → `static configurable = {}` (apply/update hooks: UNV, check docs).
- Don't subclass when a config on the stock widget does the job.

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

- `'up.name'` resolves the handler on an ancestor widget.
- Access items by key: `this.widgetMap.addButton`. Ext `reference` / `itemId` → the key in `items`.
- Button `cls : 'b-raised'` → `rendition : 'filled'`.
- Ext `header : { items }` → Panel `tools` (Grid, Scheduler, Gantt are Panels in 7.x).

## 4. ViewModel bindings → explicit state

Ext `bind : '{rowHeight}'` shared by two components → set the value explicitly on both and update from an `onChange`
handler:

```js
tools : {
    rowHeightField : { type : 'numberfield', label : 'Row height', min : 20, value : 45, onChange : 'up.onRowHeightChange' }
},
onRowHeightChange({ value }) {
    this.rowHeight = value;
}
```

In a framework target, hold the value in framework state and pass it as a prop to the Bryntum wrapper component.
ViewModel `formulas` → computed values (a calculated model field, a getter, or framework computed state). (SRC)

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
import { Popup } from '@bryntum/<product>';

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
                cancelButton : { text : 'Cancel', onClick : 'up.hide' }
            }
        },
        keyMap : { Enter : 'onSaveClick' }   // Ext dialogs often saved on Enter
    };

    onSaveClick() {
        const
            { nameField, startDateField, startTimeField } = this.widgetMap,
            fields = [nameField, startDateField, startTimeField];

        if (fields.every(field => field.isValid)) {
            // write back to the store, then:
            this.hide();
        }
    }
}

RangeEditor.initClass();
```

Full working version: `examples/scheduler-extjsmodern-vite/lib/TimeRangeEditor.js` in the extjs-migration-agent repo.
In a framework app that already has a component system (MUI, Angular Material, Vuetify, ...), use that system's dialog
for app-level dialogs instead (core skill, widget-first rule).

## 7. Rendering

- `Ext.XTemplate` / string templates → functions returning template literals. Escape user data with the
  `StringHelper.xss` tagged template. (SRC)
- Scheduler: `eventRenderer({ eventRecord, resourceRecord, renderData })` — set `renderData.cls` / `renderData.style`,
  return HTML or a DOM config. Custom layouts: see the `bryntum-styling` skill. (SRC)
- Gantt: `taskRenderer({ taskRecord, renderData })`. (SRC)
- Colors painted in renderers (`background-color : resource.color`) → `eventColor` on the event or resource (any CSS
  color) plus `eventStyle`. (SRC)
- Row CSS class (`getRowClass`) → `cls` field on the record, or a column `renderer`. (G-G)
- Grid group headers: `groupRenderer({ groupColumn, groupRowFor, isFirstColumn })` in 7.x — don't port checks like
  `data.column === data.grid.columns.first`. (SRC)
- Tooltips: the product's tooltip feature (`eventTooltip`, `taskTooltip`, `cellTooltip`) with a `template`/renderer
  function. (SRC / DOC per product)

## 8. Localization

```js
import { LocaleHelper } from '@bryntum/<product>';
import '@bryntum/<product>/locales/<product>.locale.Es';

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
toolbar widget (Gantt, Scheduler Pro — G-G). For Scheduler/Grid, confirm the StateTrackingManager wiring in the docs.

## 10. Framework targets

The migration steps are the same; only where the config lives changes. Load the matching framework skill.

| Concern | Vanilla | React | Angular | Vue 3 |
|---|---|---|---|---|
| Root widget | `new Gantt({ appendTo })` | `<BryntumGantt {...props} />` | `<bryntum-gantt [prop]="...">` | `<bryntum-gantt v-bind="config">` |
| Converted Ext config | the constructor object | a `useState`-held props object | a typed config object in the component | a typed `BryntumGanttProps` object |
| Instance access | variable / `window.gantt` | `ref.current.instance` | `@ViewChild(...).instance` | template ref `.instance` |
| Custom Bryntum classes (`Toolbar`/`Popup` subclasses) | `lib/*.js` | same classes, imported and referenced by `type` | same | same |
| ViewModel binds | handlers | React state → props | component state/inputs → bindings | refs/reactive → props |
| App dialogs outside Bryntum | `Popup` | app's component system if any, else `Popup` | same | same |

Keep Bryntum configuration inside Bryntum configs even in a framework app: toolbars, editors, menus and renderers stay
Bryntum widgets, not framework components.
