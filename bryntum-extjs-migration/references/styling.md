# Styling

Rebuild styling for Bryntum 7. Don't port Ext theme chrome. The result should look like a Bryntum 7 app that keeps the
old app's domain meaning (status colors, holidays, custom renderer markup) — not an Ext app with the old skin pasted on.

## CSS setup

Use plain CSS `@import`s as in the core `bryntum` skill. Ext Sass variables and theme packages don't carry over.

## Choosing a theme

Themes in 7.3: `svalbard`, `visby`, `stockholm`, `material3`, `fluent2`, `high-contrast`, each `-light` / `-dark`.

| Source app | Start from |
|---|---|
| Ext Classic Neptune / Triton / Crisp, or an app users know as "the blue Ext look" | `stockholm-light` |
| Ext Modern Material | `material3-light` / `material3-dark` |
| Framework target with Material UI / Angular Material / Vuetify | `material3-*` |
| Fluent / Microsoft-style host | `fluent2-*` |
| Bryntum-in-Ext app that already set a 7.x theme | keep that theme |
| Pre-7.0 combined file (`gantt.stockholm.css`) | the matching 7.x theme (`stockholm-light.css`) |
| Otherwise | `svalbard-light` (default) |

These are starting points; confirm the choice with the user when the look matters. Dark mode and runtime switching: the
`bryntum-theming` skill.

## Keep vs drop

Drop, and list each dropped rule in the report with its bucket:

- `.x-*` selectors, Ext panel/header/resizer/toolbar chrome
- colors and sizes that only mimic the old Ext theme
- rules targeting Ext DOM that no longer exists
- rules that only restate the current Bryntum theme defaults

Keep and rewrite:

- domain meaning: status colors, holidays/non-working time, priority markers, custom renderer markup
- app-specific utility rules

## Selector migration

- Ext Scheduler `sch-*` → Bryntum `b-sch-*` (e.g. `.sch-event-header` → `.b-sch-event-header`).
- Ext Gantt `sch-gantt-*` / `gnt-*` → `b-gantt-*` or `b-sch-*` depending on the element. Inspect the rendered DOM; don't
  guess.
- 7.x kebab-case renames: `.b-rownumber-cell` → `.b-row-number-cell`, `.b-timeranges` → `.b-time-ranges`,
  `.b-sch-timerange` → `.b-sch-time-range`, `.b-timeline-subgrid` → `.b-timeline-sub-grid`,
  `.b-buttongroup` → `.b-button-group`, `.b-sch-header-timeaxis-cell` → `.b-sch-header-time-axis-cell`. Full list:
  `https://bryntum.com/products/<product>/docs-llm/guide/<Product>/migration/migrate-to-new-css.md`.
- Prefer CSS variables over selector overrides. One exception: `--b-grid-header-padding` sets both the header text's
  `padding-block` and the header's `padding-inline`, so shrinking it moves the header labels sideways, out of line with
  the cells. For a shorter header, set `.b-grid-header-text { padding-block }` instead.

## Colors

Replace hard-coded Ext colors on Bryntum internals with theme tokens, or with record color fields when the color is
data-driven.

- Tokens: `--b-primary`, `--b-neutral-<n>`, `--b-text-<n>`, `--b-border-<n>`; component variables such as
  `--b-grid-header-background`, `--b-grid-header-font-weight`, `--b-grid-cell-font-size`, `--b-panel-header-background`.
- Named colors (`Core/Colors.css` in 7.3): `--b-color-red`, `-orange`, `-deep-orange`, `-amber`, `-yellow`, `-lime`,
  `-light-green`, `-green`, `-teal`, `-cyan`, `-light-blue`, `-blue`, `-indigo`, `-violet`, `-purple`, `-deep-purple`,
  `-magenta`, `-pink`, `-brown`, `-light-gray`, `-lighter-gray`, `-gray`, `-black`. Check the file in the installed
  package for the current list.
- Renderer-injected inline backgrounds → `eventColor` (event or resource field) + `eventStyle`.
- Override variables on `:root` (or a wrapper class) rather than targeting internal selectors. Some are re-declared
  per widget, so an inherited value never arrives: Stockholm sets `--b-panel-header-background`/`-color` on
  `.b-bryntum:not(.b-nothing)`, and core CSS sets `--b-panel-header-padding` on `.b-grid-base`. Set those on the widget
  itself with higher specificity: `.my-wrapper :is(.b-panel, .b-panel-header):not(.b-nothing) { … }`.
- A brand color with real meaning (the customer's primary color) may stay as a hex value, set once as a variable.

## Ext Classic density in Stockholm

Stockholm is much roomier than Crisp/Neptune (panel header 57px vs ≈36px, toolbar 65 vs 36, column header 42 vs 30,
rows 32 vs 25). A recipe that matched Crisp to within 2px, scoped to the app wrapper so popups and dialogs keep
Stockholm sizing:

```css
.my-wrapper {
    --b-toolbar-padding : 0.3em 0.6em;
    --b-button-height   : 2em;
}
.my-wrapper :is(.b-panel, .b-panel-header):not(.b-nothing) {
    --b-panel-header-padding     : 0.7em 0.8em;
    --b-panel-header-background  : #f5f5f5;   /* Crisp */
    --b-panel-header-color       : #157fcc;   /* Crisp */
    --b-panel-header-font-weight : 300;
    --b-panel-header-font-size   : 15px;
}
.my-wrapper .b-grid-header-text { padding-block : 0.55em; }
.my-wrapper .b-grid-base { --b-grid-cell-padding-inline : 10px; }   /* Crisp's cell padding */
```

Plus `rowHeight : 26` on the grid and `features : { group : { headerHeight : 30 } }`. Only aim for the old density
when the user asks for it, and list it in the report either way.

To make dialogs compact too, give the Popup a `cls` and add:

```css
.my-dialog {
    --b-text-field-input-height : 2em;
    --b-container-gap           : 0.45em;
    --b-button-height           : 2em;
}
.my-dialog .b-panel-header:not(.b-nothing) { /* the same --b-panel-header-* values as above */ }
```

## Verification

Beyond the checklist in `templates.md`: the app font is a deliberate choice (the app's own font, or Poppins per the
core skill), and no hex/rgb colors remain apart from intentional brand/domain colors.
