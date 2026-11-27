# Wrapper rules: name lists and toolkit differences

Lists used by the recipe in `SKILL.md`. Copy the ones you need into the generated module as constants.

## Wrapper-owned Bryntum names (never exposed)

Placement and identity of the Bryntum widget belong to the wrapper:

```javascript
['adopt', 'appendTo', 'insertBefore', 'insertFirst', 'renderTo', 'element', 'parent', 'owner', 'id', 'ref', 'type',
    'floating', 'positioned', 'config', 'initialConfig', 'widgetConfig']
```

## Ext instance fields (treated as Ext-owned)

Not visible on the Ext prototype, but set on instances:

```javascript
['el', 'element', 'bodyElement', 'body', 'rendered', 'initialConfig', 'config', 'ownerCt', 'parent', 'self',
    'events', 'hasListeners', 'id', 'itemId', 'xtype', 'layout', 'items', 'container', 'floatParent', 'ui']
```

Typical clashes on `Ext.Panel`: `id`, `cls`, `style`, `width`, `height`, `flex`, `hidden`, `disabled`, `listeners`,
`items`, `layout`, `html`, `data`, `tooltip`, `scrollable`, `margin`, `title`, `header`, `tbar`, `bbar`, `tools`,
`collapsible`, `border`. `title`, `tbar` and `tools` are free on `Ext.Component`.

## Ext component event names (relayed with a `widget` prefix)

The full list for both toolkits. Use all of it in the generated module, also for a Modern-only wrapper:

```javascript
['initialize', 'painted', 'erased', 'resize', 'show', 'hide', 'beforeshow', 'beforehide', 'destroy',
    'beforedestroy', 'added', 'removed', 'add', 'remove', 'beforeadd', 'beforeremove', 'render', 'beforerender',
    'afterrender', 'boxready', 'focus', 'blur', 'focusenter', 'focusleave', 'enable', 'disable', 'collapse', 'expand',
    'beforecollapse', 'beforeexpand', 'activate', 'deactivate', 'move', 'titlechange', 'hiddenchange',
    'disabledchange', 'widthchange', 'heightchange']
```

## Ext JS names → Bryntum names

Add only those that fit the wrapped product (a wrong feature name throws `Invalid type name`), and only when the
Bryntum class has no property of that name and Ext does not own it. There are three kinds:

**Feature aliases** (`featureAliases`, implemented in the example): the value becomes `features[feature]`; a setter
on a live widget sets `disabled` for a boolean and calls `setConfig()` for an object.

| Ext JS name | Feature | Products |
|---|---|---|
| `highlightWeekends` | `nonWorkingTime` | Scheduler, Gantt |
| `showRollupTasks` | `rollups` | Gantt |
| `eventEditor` | `eventEdit` | Calendar, Scheduler |
| `dayMenu` | `scheduleMenu` | Calendar |

**Plain renames** (`configAliases`): the value is passed unchanged under the Bryntum name.

| Ext JS name | Bryntum | Products |
|---|---|---|
| `snapToIncrement` | `snap` | Scheduler, Gantt |
| `emptyText` / `minValue` / `maxValue` | `placeholder` / `min` / `max` | fields |
| `iconCls` | `icon` | buttons |

Add them next to `featureAliases`, filtered the same way, then handle them in the constructor and with accessors:

```javascript
// Module scope
configAliases = { snapToIncrement : 'snap' },

// Constructor loop, before the Ext fallback
else if (configAliases[key]) {
    pending[configAliases[key]] = config[key];
}

// Accessors, after the exposed-name loop
for (const [alias, target] of Object.entries(configAliases)) {
    const cap = Ext.String.capitalize(alias);

    body[`get${cap}`] = function() {
        return this.getGantt() ? this._widget[target] : this._widgetPending?.[target];
    };
    body[`set${cap}`] = function(value) {
        (this.getGantt() || this._widgetPending)[target] = value;

        return this;
    };
}
```

**Shape-changing names** need a hand-written `getXxx`/`setXxx` and a constructor branch that converts the value:
`lockedGridConfig` → `subGridConfigs : { locked : value }` (Grid, Scheduler, Gantt), `showTodayLine` →
`features.timeRanges : { showCurrentTimeLine : value }` (Scheduler, Gantt), `allowBlank` → `required : !value`
(fields). Leave them out unless the app uses them.

Renamed configs with a changed shape, plugins to features, renderer signatures and column `dataIndex` → `field` are
in the `bryntum-extjs-migration` skill (`references/api-mapping.md`, `references/products/<product>.md`); translate
values passed into the wrapper (columns, feature configs) yourself.

## Classic toolkit differences

| | Modern | Classic |
|---|---|---|
| Ext base | `Ext.Panel` / `Ext.Component` | `Ext.panel.Panel` with `layout : 'fit'` in the body, or `Ext.Component` |
| Create the widget | `initialize` + single `painted` listener | `onBoxReady` (call `extProto.onBoxReady` first) |
| Adopted element | `bodyElement` (Panel), else `element` | `body` (Panel), else `el` |
| Body sizing kept after `adopt` | inline styles + computed `flex` | inline `width`/`height` |
| Ext configs | `getConfigurator().configs` | same where present, plus prototype members |
| Destroy | override `destroy` | same |

Not `afterRender`: it runs before Classic's first layout, so the body has no size (Gantt warns about its
`minHeight`). The default `auto` panel layout renders an `x-autocontainer-outerCt` into the body; `adopt` keeps
existing children, so it takes the height and the widget's body ends up 0px high.

```javascript
// Class body
extend : 'Ext.panel.Panel',
layout : 'fit',

onBoxReady() {
    extProto.onBoxReady.apply(this, arguments);
    this.createGantt();
}
```

### Classic page and entry module skeleton

```html
<link rel="stylesheet" href="https://bryntum.com/demos/extjs-7.4.0/build/classic/theme-triton/resources/theme-triton-all.css">
<link rel="stylesheet" href="../../build/fontawesome/css/fontawesome.css">
<link rel="stylesheet" href="../../build/fontawesome/css/solid.css">
<link rel="stylesheet" href="../../build/gantt.css">
<link rel="stylesheet" href="../../build/visby.css" data-bryntum-theme>
<script src="https://bryntum.com/demos/extjs-7.4.0/build/ext-all-debug.js"></script>
<!-- in <body> -->
<script type="module" src="app.js"></script>
```

```javascript
/* global Ext */
import './Bryntum/GanttPanel.js';

Ext.define('App.view.Main', {
    extend    : 'Ext.container.Viewport',
    requires  : ['Bryntum.GanttPanel'],
    viewModel : { data : { readOnly : false } },
    layout    : 'border',
    items     : [{
        xtype  : 'treepanel',
        region : 'west',
        width  : 220,
        split  : true
    }, {
        xtype  : 'ganttpanel',
        region : 'center',
        title  : 'Gantt',
        // Classic toolbar control bound to a Bryntum config
        tbar   : [{ xtype : 'checkbox', boxLabel : 'Read only', bind : '{readOnly}' }],
        bind   : { readOnly : '{readOnly}' },
        project : { tasks : [] }
    }]
});

Ext.application({
    name     : 'App',
    mainView : 'App.view.Main'
});
```
