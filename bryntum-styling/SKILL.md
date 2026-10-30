---
name: bryntum-styling
description: >
  Custom styling of Bryntum Scheduler event bars, any tier incl. the former Scheduler Pro (eventRenderer layouts, event bar
  DOM structure, padding CSS variables, sticky content) and widget `rendition` (built-in visual
  variants of buttons, text fields and tooltips, plus the form look). Use alongside the `bryntum`
  skill. Trigger on "eventRenderer", "style the event bars", "customize event content", "two-line
  events", "event bar padding", content not filling / overflowing the bar, "button style",
  "primary button", "filled button", "rendition", "outlined field", "form layout".
  For picking themes or dark mode use `bryntum-theming` instead.
metadata:
  tags: bryntum, styling, eventrenderer, event-bar, scheduler, schedulerpro, css, rendition, button, form
---

## Version check — do this first

Detect the version as the `bryntum` skill's Version check says (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md). Every code block in this file is 8.0 code. 7.x → also load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-styling/references/v7.md).

---

## Event bar structure

```html
<div class="b-sch-event-wrap">      <!-- renderData.wrapperCls classes land here -->
    <div class="b-sch-event">       <!-- renderData.cls classes land here -->
        <div class="b-sch-event-content">
            <!-- eventRenderer output goes here -->
        </div>
    </div>
</div>
```

- `.b-sch-event-content` already has padding. Adjust it with `--b-sch-event-padding-inline` (horizontal mode) or `--b-sch-event-padding-block` (vertical mode) on the wrapper, e.g. `.b-sch-event-wrap { --b-sch-event-padding-inline: 1em; }`.
- Event content is sticky by default on Business and Enterprise (`stickyEvents` is a Business feature; on Community content is not sticky and the config is ignored) — kept in view while scrolling the time axis, so it doesn't stretch to fill the bar. For layouts that should fill it (multi-line, stacked), set `features: { stickyEvents: false }`.
- `eventRenderer({ eventRecord, renderData })` can return a DOM config array. To suppress the default icon (e.g. you render it yourself, or want none), set `renderData.iconCls = ''`. `null` also works in JS, but the TypeScript type is `string | DomClassList`, so `null` fails under `strict`.

Worked two-line layout (time + icon header, bold name below) with its CSS: `references/two-line-event-bar.md`. Scope your CSS with `.b-sch-event-wrap …`; for a component-level selector use `.b-scheduler-base`, which matches every Scheduler.

## Widget renditions

Buttons, text fields and tooltips ship several looks. Use the `rendition` config rather than custom CSS or invented class names; the value becomes a class (`b-button-tonal`, `b-text-field-outlined`) that the theme styles. Tooltips are the exception: `'rich'` adds `b-rich-tooltip`.

| Widget | `rendition` values | Default |
|---|---|---|
| `Button` (and subclasses) | `'elevated'` raised with a shadow · `'filled'` primary colour · `'tonal'` faded primary · `'outlined'` border, pale/transparent fill · `'text'` transparent, text only | theme-dependent (`text` in Svalbard) |
| `ButtonGroup` | the Button values, plus `'padded'` and `'padded-filled'`; applied to every button in the group | as Button |
| `TextField`, `TextAreaField`, `NumberField`, `DateTimeField` and subclasses (Combo, DateField, TimeField, DurationField, …) | `'outlined'` · `'filled'` | `'outlined'` |
| `Tooltip` | `'plain'` · `'rich'` (title + body chrome) | auto: rich when the content has a header, items or nested markup, else plain |

Nothing else takes `rendition` (not Panel, Toolbar, Grid columns or Menu items); style those with CSS variables. Set it on the primary action only, e.g. `{ type : 'button', text : 'Save', rendition : 'tonal' }` (`'filled'` for the strongest emphasis).

Making a panel of fields look like a form (width, `labelPosition: 'align-before'`, tonal primary button): `references/form-look.md`. Setting `defaults : { rendition : 'filled' }` on the panel applies it to every top-level field. Composite fields don't pass it down: for `DateTimeField` also set `dateField : { rendition : 'filled' }, timeField : { rendition : 'filled' }`. For a dialog form use a `Popup` (`modal : true, centered : true`) instead of a Panel.

Tooltips: `features.eventTooltip` takes `rendition : 'rich'` directly. Set the title with `tip.title = …` inside its `template`. The top-level DomConfig returned from `template` is merged into the tooltip's content element (a `tag`/`class` on it was dropped in testing), so put any element you need to style on a child.

## Related

- Themes, dark mode, and theme-level CSS variables: `bryntum-theming` skill.
- CSS rules of thumb: see the `bryntum` skill.
