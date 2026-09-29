# Styling

Rebuild styling for Bryntum 7. Don't port Ext theme chrome. The result should look like a Bryntum 7 app that keeps the
old app's domain meaning (status colors, holidays, custom renderer markup) — not an Ext app with the old skin pasted on.

## CSS setup

Plain CSS, as in the core `bryntum` skill (no SASS; Ext Sass variables and theme packages don't carry over):

```css
@import "@bryntum/<product>/fontawesome/css/fontawesome.css";
@import "@bryntum/<product>/fontawesome/css/solid.css";
@import "@bryntum/<product>/<product>.css";        /* structural, required */
@import "@bryntum/<product>/stockholm-light.css";  /* one theme */
/* app rules last */
```

Only when migrating an official Bryntum demo: add `@bryntum/demo-resources` (its `scss/example.scss` needs `sass`) and
`DemoHeader`, as in the extjs-migration-agent examples.

## Choosing a theme

Themes in 7.3: `svalbard`, `visby`, `stockholm`, `material3`, `fluent2`, `high-contrast`, each `-light` / `-dark`.

| Source app | Start from |
|---|---|
| Ext Classic Neptune / Triton / Crisp, or an app users know as "the blue Ext look" | `stockholm-light` |
| Ext Modern Material | `material3-light` / `material3-dark` |
| Framework target with Material UI / Angular Material / Vuetify | `material3-*` |
| Fluent / Microsoft-style host | `fluent2-*` |
| Bryntum-in-Ext demo that already set a theme (`data-bryntum-theme` link) | keep that theme |
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
  `.b-buttongroup` → `.b-button-group`. Full list:
  `https://bryntum.com/products/<product>/docs-llm/guide/<Product>/migration/migrate-to-new-css.md`.
- Prefer CSS variables over selector overrides.

## Colors

Replace hard-coded Ext colors on Bryntum internals with theme tokens, or with record color fields when the color is
data-driven.

- Tokens: `--b-primary`, `--b-neutral-<n>`, `--b-text-<n>`, `--b-border-<n>`; component variables such as
  `--b-grid-header-background`, `--b-grid-header-font-weight`, `--b-grid-cell-font-size`, `--b-panel-header-background`.
- Named colors in 7.3: `--b-color-red`, `-orange`, `-amber`, `-lime`, `-green`, `-teal`, `-cyan`, `-blue`, `-indigo`,
  `-purple`, `-magenta`, `-pink`, `-brown`, `-gray`, `-black`.
- Renderer-injected inline backgrounds → `eventColor` (event or resource field) + `eventStyle`.
- Override variables on `:root` (or a wrapper class) rather than targeting internal selectors.
- A brand color with real meaning (the customer's primary color) may stay as a hex value, set once as a variable.

## Verification

- no `.x-*` selectors remain; no Ext stylesheet is loaded
- icons (FontAwesome) render in the dev server, not just the build
- the app font is intentional (the app's own font, or Poppins per the core skill when there is none)
- no rules that only restate theme defaults; no hex/rgb colors except intentional brand/domain colors
