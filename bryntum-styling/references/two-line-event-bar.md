# Two-line event bar layout

A header row with start time + icon, and a bold name below. `stickyEvents: false` and clearing `renderData.iconCls` are the non-obvious parts. For time and name only, drop the icon and the header wrapper (put the time span directly in the array) and skip `iconCls`.

```js
const scheduler = new Scheduler({
    features : {
        stickyEvents : false   // lets event content stretch to fill the bar
    },

    eventRenderer({ eventRecord, renderData }) {
        const { iconCls } = renderData;

        // The icon is rendered in the header below, suppress the default one
        renderData.iconCls = '';   // '' rather than null: the TS type is string | DomClassList

        return [
            {
                class    : 'b-event-header',
                children : [
                    {
                        tag   : 'span',
                        class : 'b-event-time',
                        text  : DateHelper.format(eventRecord.startDate, 'h:mm A').toLowerCase()
                    },
                    iconCls?.length ? { tag : 'i', class : iconCls } : null
                ]
            },
            { class : 'b-event-name', text : eventRecord.name }
        ];
    }
});
```

```css
.b-sch-event-wrap .b-sch-event-content {   /* not .b-scheduler: Scheduler Pro's root lacks that class */
    flex-direction               : column;
    align-items                  : stretch;
    justify-content              : center;
    gap                          : 0.2em;

    --b-sch-event-padding-inline : .75em;
}

.b-event-header {
    font-weight     : 400;
    display         : flex;
    align-items     : center;
    justify-content : space-between;
    font-size       : 0.8em;
}

.b-event-name {
    font-weight : 700;
}

.b-event-time,
.b-event-name {
    overflow      : hidden;
    text-overflow : ellipsis;
    white-space   : nowrap;
}
```

Events supply `iconCls` (e.g. `'fa fa-user'`) in their data. Give rows room for two lines (e.g. `rowHeight : 75`).

- Event text inherits the theme's event weight (`--b-sch-event-font-weight`, often 600), so set both weights explicitly or the name won't stand out.
- The bold name line usually sets the minimum bar width, not the time line (about 150px for a 14-character name at default size; a `HH:mm – HH:mm` line needs roughly 90–100px). Size `tickSize` from your shortest event × longest name, e.g. 220 per hour for 45-minute events, and accept horizontal scroll.
