# Example output: Calendar wrapper (Modern toolkit)

This is an example of what to **generate** for one Bryntum class by following `SKILL.md`, not a library to import.
It is the `Bryntum/CalendarPanel.js` module of the Calendar `extjsmodern` demo: an `Ext.Panel`
(`xtype : 'calendarpanel'`) around a Bryntum Calendar. For another class change the import, `WidgetClass`, the names
(`xtype`, `getCalendar`, `createCalendar`), the class-body defaults and the Ext JS aliases that fit the product.
For the Classic toolkit change the base, the create hook and the adopted element as listed in
`wrapper-rules.md` (Classic toolkit differences).

```javascript
/* global Ext */
import { Calendar } from '../../../build/calendar.module.js';

/**
 * Ext JS Modern panel wrapping a Bryntum Calendar: `xtype : 'calendarpanel'`.
 *
 * Every settable Calendar config and property is found by introspecting the Calendar class and gets a
 * `getXxx()`/`setXxx()` pair, so it works as an Ext config, in `bind` and at runtime. Names Ext owns (`title`, `tbar`,
 * `cls`, `listeners`, ...) stay Ext's; their Calendar values go in `widgetConfig`. Calendar events are fired as Ext
 * events, lower-cased, with this component as the first argument.
 */

const
    // Ext.define stamps `$name` on functions in a class body, so the Bryntum class is only referenced from here
    WidgetClass       = Calendar,
    extProto          = Ext.Panel.prototype,
    extConfigs        = Ext.Panel.getConfigurator().configs,
    // Placement and identity of the Calendar are owned by the wrapper
    wrapperOwned      = new Set([
        'adopt', 'appendTo', 'insertBefore', 'insertFirst', 'renderTo', 'element', 'parent', 'owner', 'id', 'ref',
        'type', 'floating', 'positioned', 'config', 'initialConfig', 'widgetConfig'
    ]),
    // Ext instance fields that are not on the Ext prototype
    extInstanceFields = new Set([
        'el', 'element', 'bodyElement', 'body', 'rendered', 'initialConfig', 'config', 'ownerCt', 'parent', 'self',
        'events', 'hasListeners', 'id', 'itemId', 'xtype', 'layout', 'items', 'container', 'floatParent', 'ui'
    ]),
    // Ext component events of both toolkits; a Calendar event of the same name is fired as `widget<name>`
    extEventNames     = new Set([
        'initialize', 'painted', 'erased', 'resize', 'show', 'hide', 'beforeshow', 'beforehide', 'destroy',
        'beforedestroy', 'added', 'removed', 'add', 'remove', 'beforeadd', 'beforeremove', 'render', 'beforerender',
        'afterrender', 'boxready', 'focus', 'blur', 'focusenter', 'focusleave', 'enable', 'disable', 'collapse',
        'expand', 'beforecollapse', 'beforeexpand', 'activate', 'deactivate', 'move', 'titlechange', 'hiddenchange',
        'disabledchange', 'widthchange', 'heightchange'
    ]),
    // Ext JS calendar names mapped to Calendar features (bryntum-extjs-migration skill, api-mapping.md). Plain renames
    // would go in a `configAliases` map, the Calendar has none
    featureAliases    = {
        eventEditor : 'eventEdit',
        dayMenu     : 'scheduleMenu'
    },
    // String values of these keys name a ViewController method (`eventRenderer : 'renderEvent'`)
    callbackKeyRe     = /(Renderer|renderer|Fn|Template|Tpl|Handler|handler)$/,
    dataKeyRe         = /(data|Data|store|Store|project|crudManager)$/,
    identifierRe      = /^[A-Za-z_$][\w$]*$/,
    isPlainObject     = value => value !== null && typeof value === 'object' && value.constructor === Object,
    isExtOwned        = name => {
        const cap = Ext.String.capitalize(name);

        return extInstanceFields.has(name) || name in extConfigs ||
            [name, `get${cap}`, `set${cap}`].some(n => n in extProto);
    },
    names             = new Set(),
    exposed           = [],
    clashes           = [];

// All configs of the class hierarchy ($meta.configs chains to the superclass), plus plain `set foo()` accessors
for (const name in WidgetClass.$meta.configs) {
    names.add(name);
}
for (let proto = WidgetClass.prototype; proto && proto !== Object.prototype; proto = Object.getPrototypeOf(proto)) {
    for (const name of Object.getOwnPropertyNames(proto)) {
        Object.getOwnPropertyDescriptor(proto, name).set && names.add(name);
    }
}
for (const name of names) {
    if (/^[a-z]/.test(name) && !/^(is|has)[A-Z]/.test(name) && !wrapperOwned.has(name)) {
        (isExtOwned(name) ? clashes : exposed).push(name);
    }
}
for (const alias in featureAliases) {
    if (names.has(alias) || isExtOwned(alias)) {
        delete featureAliases[alias];
    }
}

// Ext maps these to webkit* names that Chrome no longer fires, which would hang Bryntum animations
delete Ext.dom.Element.prototype.eventMap?.transitionend;
delete Ext.dom.Element.prototype.eventMap?.animationstart;
delete Ext.dom.Element.prototype.eventMap?.animationend;

const body = {
    extend : 'Ext.Panel',
    xtype  : 'calendarpanel',
    cls    : 'bryntum-calendar',

    // Calendar defaults for every instance, declared like any Ext class-body default. `tier` is only needed to run a
    // package at a lower tier than its own (this demo ships in the Core tier); remove it otherwise
    tier : 'core',

    /**
     * Calendar names exposed as getXxx/setXxx and Ext configs
     * @readonly
     */
    bryntumProperties : exposed,

    /**
     * Calendar names the Ext class owns; pass these in `widgetConfig`
     * @readonly
     */
    bryntumClashes : clashes,

    constructor(config = {}) {
        const
            me      = this,
            extCfg  = {},
            pending = {};

        // Class-body defaults, also those of Ext subclasses
        for (const name of exposed) {
            if (me[name] !== undefined) {
                pending[name] = me[name];
            }
        }

        config.widgetConfig && Ext.merge(pending, config.widgetConfig);

        for (const key in config) {
            if (key === 'widgetConfig') {
                continue;
            }
            if (exposed.includes(key)) {
                pending[key] = config[key];
            }
            else if (featureAliases[key]) {
                pending.features = { ...pending.features, [featureAliases[key]] : config[key] };
            }
            else {
                extCfg[key] = config[key];
            }
        }

        me._widgetPending = pending;

        // callParent() needs Function.caller, which module code does not have
        extProto.constructor.call(me, extCfg);
    },

    initialize() {
        extProto.initialize.call(this);

        this.on({
            painted : () => this.createCalendar(),
            single  : true
        });
    },

    /**
     * The Bryntum Calendar, or `null` until this component is painted
     * @returns {Calendar.view.Calendar}
     */
    getCalendar() {
        const widget = this._widget;

        return widget && !widget.isDestroyed ? widget : null;
    },

    /**
     * Replaces ViewController method names in callback-style configs with functions calling them
     * @private
     */
    resolveCallbacks(key, value) {
        if (typeof value === 'string') {
            return key && callbackKeyRe.test(key) && identifierRe.test(value)
                ? (...args) => Ext.callback(value, null, args, 0, this)
                : value;
        }
        if (Array.isArray(value)) {
            return value.map(item => this.resolveCallbacks(null, item));
        }
        if (isPlainObject(value) && !(key && dataKeyRe.test(key))) {
            const result = {};

            for (const k in value) {
                result[k] = this.resolveCallbacks(k, value[k]);
            }

            return result;
        }

        return value;
    },

    /**
     * Creates the Calendar on the panel's body element, which the Ext layout sizes
     * @private
     */
    createCalendar() {
        const me = this;

        if (me._widget || me.destroyed) {
            return;
        }

        const
            { dom }   = me.bodyElement,
            { style } = dom,
            extStyle  = [...style].map(prop => [prop, style.getPropertyValue(prop), style.getPropertyPriority(prop)]),
            // Ext's layout gives the body `flex : 1` through `.x-container-body-el`, which goes with the `x-` classes
            extFlex   = getComputedStyle(dom).flex,
            config    = me.resolveCallbacks(null, me._widgetPending),
            cls       = [...dom.classList].filter(c => !c.startsWith('x-')).join(' ');

        // `adopt` clears className and inline style. Ext's `x-` classes stay off to keep Ext styling out
        config.adopt = dom;
        cls && (config.cls = config.cls ? `${config.cls} ${cls}` : cls);

        me._widgetPending = null;
        me._widget = new WidgetClass(config);

        // Ext's inline layout styles and its class-based flex, then a font size not inherited from Ext's (12px in
        // Ext Material). Only where Bryntum set none
        for (const [prop, value, priority] of extStyle) {
            style.getPropertyValue(prop) || style.setProperty(prop, value, priority);
        }
        style.getPropertyValue('flex') || style.setProperty('flex', extFlex);
        style.getPropertyValue('font-size') || style.setProperty('font-size', 'initial');

        me._widget.on({
            catchAll : me.relayCalendarEvent,
            thisObj  : me,
            prio     : 100
        });

        me.fireEvent('widgetcreate', me, me._widget);
    },

    /**
     * Fires a Calendar event on this component as `(component, event)`; `false` vetoes `before*` events
     * @private
     */
    relayCalendarEvent(event) {
        // `eventName`, as `type` keeps the first name when Bryntum re-triggers one object (`drag` in external drops)
        const
            type = event.eventName.toLowerCase(),
            name = extEventNames.has(type) ? `widget${type}` : type;

        if (this.hasListeners?.[name]) {
            return this.fireEvent(name, this, event);
        }
    },

    destroy() {
        const widget = this.getCalendar();

        this._widget = this._widgetPending = null;

        // Calendar first, while the Ext DOM it lives in still exists
        widget?.destroy();

        extProto.destroy.call(this);
    }
};

// getXxx/setXxx for every exposed name: the pending config before the Calendar exists, the Calendar afterwards
for (const name of exposed) {
    const cap = Ext.String.capitalize(name);

    body[`get${cap}`] = function() {
        return this.getCalendar() ? this._widget[name] : this._widgetPending?.[name];
    };

    body[`set${cap}`] = function(value) {
        (this.getCalendar() || this._widgetPending)[name] = this.resolveCallbacks(name, value);

        return this;
    };
}

for (const [alias, feature] of Object.entries(featureAliases)) {
    const cap = Ext.String.capitalize(alias);

    body[`get${cap}`] = function() {
        return this.getCalendar() ? this._widget.features[feature] : this._widgetPending?.features?.[feature];
    };

    body[`set${cap}`] = function(value) {
        const instance = this.getCalendar()?.features[feature];

        if (!this._widget) {
            this._widgetPending.features = { ...this._widgetPending.features, [feature] : value };
        }
        else if (instance && typeof value === 'boolean') {
            instance.disabled = !value;
        }
        else if (instance && isPlainObject(value)) {
            instance.setConfig(value);
        }

        return this;
    };
}

Ext.define('Bryntum.CalendarPanel', body);
```

## Page and entry module skeleton

`index.html`: Ext JS as a classic script with its theme, Bryntum CSS (product, theme, Font Awesome), then the app as
a module. Paths assume the page sits two levels below the product folder, next to `build/`:

```html
<!DOCTYPE html>
<html lang="en" style="color-scheme: light">
<head>
    <meta charset="UTF-8">
    <link rel="icon" href="data:,">
    <link rel="stylesheet" href="https://bryntum.com/demos/extjs-7.4.0/build/modern/theme-material/resources/theme-material-all-debug.css">
    <link rel="stylesheet" href="../../build/fontawesome/css/fontawesome.css">
    <link rel="stylesheet" href="../../build/fontawesome/css/solid.css">
    <link rel="stylesheet" href="../../build/calendar.css">
    <link rel="stylesheet" href="../../build/visby.css" data-bryntum-theme>
    <script src="https://bryntum.com/demos/extjs-7.4.0/build/ext-modern-all-debug.js"></script>
</head>
<body>
    <script type="module" src="app.js"></script>
</body>
</html>
```

`app.js`: import the wrapper first, define the ViewModel, ViewController and main view in the module (no
`Ext.Loader` paths needed), then start the application. An Ext control in the panel header binds to a Bryntum config
through the ViewModel:

```javascript
/* global Ext */
import './Bryntum/CalendarPanel.js';

Ext.define('App.view.MainController', {
    extend : 'Ext.app.ViewController',
    alias  : 'controller.main',

    // Clicking a date header drills into day view, keep the ViewModel in sync
    onModeChange(cmp) {
        this.getViewModel().set('mode', cmp.getMode());
    }
});

Ext.define('App.view.Main', {
    extend     : 'Ext.Container',
    xtype      : 'app-main',
    requires   : ['Bryntum.CalendarPanel'],
    controller : 'main',
    viewModel  : { data : { mode : 'week' } },
    layout     : 'fit',
    items      : [{
        xtype  : 'calendarpanel',
        title  : 'Calendar',
        header : {
            items : [{
                xtype   : 'selectfield',
                label   : 'Mode',
                bind    : '{mode}',
                options : ['day', 'week', 'month'].map(value => ({ text : value, value }))
            }]
        },
        bind         : { mode : '{mode}' },
        widgetConfig : { tbar : { items : { modeSelector : false } } },
        date         : new Date(2026, 10, 25),
        project      : { loadUrl : 'data/data.json', autoLoad : true },
        listeners    : { activeitemchange : 'onModeChange' }
    }]
});

Ext.application({
    name     : 'App',
    mainView : 'App.view.Main'
});
```
