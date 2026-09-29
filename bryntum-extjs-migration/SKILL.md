---
name: bryntum-extjs-migration
description: >
  Migrate an Ext JS application to Bryntum 7 — legacy Ext Scheduler / Gantt (`Sch.*`, `Gnt.*`),
  Bryntum components wrapped in an Ext JS shell, or plain Ext grids and tree grids
  (`Ext.grid.Panel`, `Ext.tree.Panel`) — targeting vanilla JS, React, Angular, or Vue. Use alongside
  the `bryntum` skill. Trigger on phrases like "migrate from Ext JS", "port our Ext app", "replace
  Sencha", "Ext JS end of life", "Sch.panel", "Gnt.panel", "Ext.grid.Panel to Bryntum", or any
  Ext JS codebase being moved to Bryntum.
metadata:
  tags: bryntum, extjs, sencha, migration, scheduler, gantt, grid, schedulerpro
---

# Ext JS → Bryntum migration

Converts an Ext JS app (Classic or Modern toolkit) into a Bryntum 7 app with no Ext JS left, or — when the user wants
an incremental path — replaces the Ext components first and removes the Ext shell later.

Always load the core `bryntum` skill too, plus the framework skill for the target (`bryntum-react`,
`bryntum-angular`, `bryntum-vue`, `bryntum-vanilla`). If the migration has a real backend, load `bryntum-crud`;
if it customizes the event/task editor, load `bryntum-editor`.

Reference files (read only what the current app needs — don't load them all up front):

| File | Use it for |
|---|---|
| `references/api-mapping.md` | Ext class / config / plugin / column / method → Bryntum, **and** things with no equivalent |
| `references/data-migration.md` | Models, stores, field mapping vs data conversion, CrudManager, Gantt data changes |
| `references/patterns.md` | Idiom rewrites: app shell, classes, toolbars, dialogs, editors, renderers, locales, undo/redo, framework targets |
| `references/styling.md` | Theme choice, dropping Ext chrome, selector renames, color tokens |
| `references/products/<product>.md` | Product-specific rules and support level: `grid`, `scheduler`, `schedulerpro`, `gantt` |
| `references/templates.md` | `MIGRATION_PLAN.md` / `MIGRATION_REPORT.md` formats and the verification checklist |

Finished, verified migrations (Bryntum's own Ext demos) live in the
[extjs-migration-agent examples](https://github.com/bryntum/extjs-migration-agent/tree/main/examples). Use the closest
one as a structural model, not as a feature checklist.

---

## Rules (always)

1. **Never invent a name.** Every class, config, feature, field, column type and CSS class must come from the
   references here, the Bryntum MCP docs (`mcp__bryntum__search_bryntum_docs` with the installed `version`), the
   Bryntum docs, or the installed package's source. If none confirm it, it is **unmapped**: leave a
   `// MIGRATION:` comment where it was and list it in the report.
2. **Never edit the original app in place.** Write to a new sibling folder, or to a new git branch if the user
   prefers in-repo migration. Never `rm -rf` anything.
3. **Never silently drop behavior.** Anything removed, changed or unmapped goes in the report with a reason.
4. **Keep the data working.** Every record, relation and visible behavior the original depends on must load.
5. **Stop and ask** if more than a third of the inventoried items are unmapped, or if the app depends on a backend
   you can't run and the user hasn't said how to handle it.
6. **Don't rebuild Ext.** Bryntum is not a drop-in Ext replacement. Map to idiomatic Bryntum (features, keyed
   `items`, `widgetMap`, stores/project) — never write a compatibility layer that emulates `Ext.*` APIs.

### Lookup order

1. `references/` in this skill (corrected, verified against Bryntum 7.3.7).
2. Bryntum MCP `search_bryntum_docs` for the installed version, or `bryntum.com/products/{product}/docs/`.
3. The installed package source (`node_modules/@bryntum/{product}/`).
4. Nothing found → unmapped. Official Ext-migration guides and older blog posts contain outdated v5/v6 advice; the
   "Outdated advice" section of `references/api-mapping.md` lists the known traps.

---

## Step 0: Ask before starting

Ask these together (skip any the user already answered). Record the answers at the top of `MIGRATION_PLAN.md`.

1. **Trial or licensed package?** Trial: `npm install @bryntum/{product}@npm:@bryntum/{product}-trial` (public npm,
   watermark). Licensed: Bryntum's private registry (`npm.bryntum.com`) or a local distribution build. The public-npm
   `@bryntum/{product}` without the registry is a placeholder — never install it. Don't pick silently; if licensed
   access fails, do all non-install work and mark verification **blocked** rather than switching to trial.
2. **Target stack?** Vanilla JS + Vite (closest to Ext's config-object style, the default), React, Angular, or Vue.
   Also: TypeScript or JavaScript (default TypeScript for new apps, per the core skill; match the source if it is a
   JS codebase the team will keep maintaining).
3. **Big-bang or incremental?** Big-bang: new app, Ext removed. Incremental: first replace Ext Scheduler/Gantt/grids
   with Bryntum widgets hosted inside the existing Ext shell (a "Bryntum embedded in Ext" state), then remove the shell
   screen by screen. Incremental needs the UMD/module build loaded next to Ext; confirm the team accepts that interim.
4. **Backend?** Keep the existing server and JSON shape (map fields in models), or change the server/data to Bryntum's
   format. See `references/data-migration.md`.
5. **Scope?** Whole app, or named screens/views. For large apps, migrate one screen end to end first as a pilot.

---

## Step 1: Classify the source

| Type | How to tell | What changes |
|---|---|---|
| **A. Legacy Ext Scheduler / Gantt** | `Sch.*`, `Gnt.*` classes, `ptype` plugins, PascalCase data (`StartDate`, `ResourceId`) | Everything: shell, configs, plugins, columns, renderers, **data** |
| **B. Bryntum embedded in Ext** | `bryntum.<product>.*` globals or wrapper classes (`Bryntum.SchedulerPanel`), an Ext panel around a Bryntum widget, camelCase data | Only the Ext shell and surrounding Ext widgets. The Bryntum configs inside are already right — copy them and re-check against 7.x defaults. Data usually copies as is |
| **C. Plain Ext components** | `Ext.grid.Panel`, `Ext.tree.Panel`, Ext stores/models, no Bryntum at all | Map to Bryntum `Grid` / `TreeGrid` (see `references/products/grid.md`). Non-grid Ext UI (forms, charts, routing) maps to the target framework or Bryntum widgets |

A real app is often a mix: note the type per screen.

Also identify: toolkit (Classic / Modern), Ext version, Sencha Cmd vs other build, `Ext.app.Application` +
controllers + ViewModels, backend proxies, locales, custom `Ext.override`s, and anything else in the §8 "No
equivalent" list of `references/api-mapping.md`.

**Out of scope for this skill** — flag, don't migrate by guesswork: Ext Charts, Ext Pivot, `Ext.calendar` (map to
Bryntum Calendar only with the core skill + docs, and mark it limited in the report), Ext Direct, routing/history,
and private Ext/Sch/Gnt internals.

---

## Step 2: Inventory and plan

Read every file in scope. Write `MIGRATION_PLAN.md` (format in `references/templates.md`) with one row per item:

| Ext item | Where (file:line) | Bryntum target | Source of mapping | Status (mapped / changed / unmapped) |

Checklist: app shell (`Ext.application`, controllers, `Viewport`, `requires`); every `Ext.define` (views, models,
stores, plugins, overrides); root widget configs; every `plugins` entry; every column (`xtype`, `dataIndex`,
`filter`, editor); renderers and templates (`eventRenderer`, `taskBodyTemplate`, `getRowClass`, `*Tpl`); toolbars,
buttons and their handlers; dialogs and forms; stores, models, proxies and the data they load; product data
structures (calendars, baselines, assignments, dependencies); ViewModel bindings; localization; undo/redo; custom CSS.

Show the user the plan summary (counts of mapped / changed / unmapped, plus the unmapped list) before writing code
on anything bigger than a single screen.

---

## Step 3: Scaffold

Follow the core `bryntum` skill for install, CSS, sizing and Vite `optimizeDeps`, and the framework skill for the
component wrapper. Migration-specific points:

- **CSS: plain CSS `@import`**, as the core skill requires — FontAwesome, `{product}.css`, one theme, then app
  rules. No SASS. Remove every Ext script and stylesheet (`ext-all.js`, `bootstrap.js`, `app.json`, theme packages).
- **Theme**: pick the closest to the app's look — `stockholm-light` for classic Ext "Neptune/Triton" apps,
  `material3-*` for Ext Modern Material, else `svalbard-light`. See `references/styling.md`.
- **Keep the app's identity**: page title, meta description, favicon, visible labels.
- **Bryntum's own demos only**: when migrating an official Bryntum Ext demo, use `@bryntum/demo-resources` and
  `DemoHeader` like the examples repo does (its stylesheet is `scss/example.scss`, so this is the one case that adds
  `sass`). Never add them to a customer app.

Checkpoint: an empty root widget renders with the theme applied.

---

## Step 4: Data

Pick **one** strategy per app and record it in the plan (details in `references/data-migration.md`):

1. **Field mapping** — keep the server/JSON; declare fields with `dataSource` on models or store `fields`. Default
   when a backend exists that the user doesn't want to change.
2. **Data conversion** — convert to Bryntum's camelCase fields and structures. Required for Gantt structural changes
   (calendars, `TaskType`, baselines) that can't be expressed as field mappings. For static JSON, use a checked-in
   conversion script.

Count the important records before and after, and record the counts in the report. Ext proxies → CrudManager
(Scheduler, Scheduler Pro, Gantt) or store `readUrl`/`createUrl`/… (Grid); anything beyond plain AJAX/JSON/REST is
flagged for a human.

---

## Step 5: Convert, item by item

Work through the plan in this order, checking each item against `references/api-mapping.md` then
`references/products/<product>.md`, then docs:

1. **Root widget and data hub** — `new Scheduler/Gantt/Grid(...)` (or the framework component) with its stores or
   `project`. Share one store/project between related widgets.
2. **Configs** — for each: unchanged? renamed? now a feature? obsolete? Set `barMargin`, `rowHeight`, `viewPreset`,
   `startDate`/`endDate` explicitly; defaults differ from Ext.
3. **Plugins → `features`**, only when the mapping is confirmed.
4. **Columns** — `header` → `text`, `dataIndex` → `field`, `xtype` → `type`, editors → `editor`.
5. **Renderers and templates** — functions returning template literals; escape user data with the `StringHelper.xss`
   tagged template; convert Ext (PHP-style) date tokens to `DateHelper` (moment-style) tokens.
6. **Toolbars, menus, dialogs** — keyed `items` + `widgetMap`; `handler` → `onClick`/`onAction` (`'up.method'`);
   custom Ext event editors → the built-in `eventEdit`/`taskEdit` customized via `items`; other dialogs → a `Popup`
   subclass (vanilla) or the host framework's dialog (React/Angular/Vue app with its own component system — see the
   widget-first rule in the core skill).
7. **Models and editors** — subclass the product model only for fields the app really uses.
8. **App logic** — controllers and ViewModel `bind` → explicit handlers (`onChange`, `selectionChange`) in vanilla,
   or framework state (React state, Angular signals/inputs, Vue refs) in a framework target.
9. **Extras only if the source had them** — localization, undo/redo, side panels, RTL.
10. **Styling** — rebuild for Bryntum 7; don't port Ext chrome. Keep only colors/styles with domain meaning.

---

## Step 6: Verify (do not skip)

Full checklist in `references/templates.md`. At minimum:

1. `npm run build` passes **and** the dev server runs with no console errors or warnings (Vite's dev server enforces
   file-serving rules the build doesn't — icons/fonts can 404 only in dev).
2. Open the app in a headless browser (Playwright) and **drive it like a user** — real clicks, typing, right-clicks.
   Helper APIs like `showContextMenuFor()` skip logic and prove nothing. Check: store/project counts match the data
   counts; every planned column and feature exists; toolbar buttons have their effect; editors/dialogs open with the
   right title and values, validate, save and cancel; no blank menu items; test targets are on screen.
3. Compare against the original (screenshots or the running Ext app) and list visible differences.
4. Fix and repeat, at most 5 loops. If it still fails, stop and report what fails.

For a vanilla target, expose the root widget and data hub on `window` so checks can inspect them. For frameworks, use
the wrapper's `instance` ref.

---

## Step 7: Report

Write `MIGRATION_REPORT.md` (format in `references/templates.md`): source/target/status and package line; file-by-file
table; config mapping table with the source of each mapping; data counts before/after and structural changes;
unmapped/dropped items with reasons (including dropped CSS rules); behavior differences; verification results.

Then follow the core skill's "Verify" hand-off: leave the dev server running, and suggest real Bryntum features
the old Ext app didn't have (name them and link to `https://bryntum.com/products/{product}/docs/`).

---

## Traps

- Official Ext → Bryntum guides predate 7.x in places (`tplData`, `extraItems` + `index`, `showResourceField`,
  assignment `taskId`/`resourceId`, `material2` themes, `.b-rownumber-cell`). Trust `references/api-mapping.md`.
- 7.x renamed CSS classes to kebab-case (`.b-timeline-subgrid` → `.b-timeline-sub-grid`). Translate any copied
  selectors.
- `items : { someBuiltIn : true }` on a menu/editor **replaces** the built-in item config (blank menu text). Only
  override the properties you change, or set `false`/`null` to remove.
- Setting `eventEdit`/`taskEdit` to `false` removes the `beforeEventEdit` hook — to use a custom dialog, keep the
  feature and return `false` from the hook (see `bryntum-editor`).
- A 7.x store `grouper.field` must be a field name — a function throws. Use a calculated field for computed groups.
- Don't assume a mapping from one product holds for another (Scheduler `eventRenderer` vs Gantt `taskRenderer`).
- When switching trial ↔ licensed, move `package-lock.json` and `node_modules/` aside first; the lockfile silently
  keeps the old source.
