---
name: bryntum-extjs-migration
description: >
  Migrate an Ext JS application to Bryntum 8 (or 7.x) — legacy Ext Scheduler / Gantt (`Sch.*`, `Gnt.*`),
  Bryntum components wrapped in an Ext JS shell, or plain Ext grids and tree grids
  (`Ext.grid.Panel`, `Ext.tree.Panel`) — targeting vanilla JS, React, Angular, or Vue. Use alongside
  the `bryntum` skill. Trigger on phrases like "migrate from Ext JS", "port our Ext app", "replace
  Sencha", "Ext JS end of life", "Sch.panel", "Gnt.panel", "Ext.grid.Panel to Bryntum", or any
  Ext JS codebase being moved to Bryntum. Also use it whenever Ext JS habits appear in Bryntum code
  (`store.first()`, `component.down()`, `store.get(id)`, `Ext.getCmp`, `Ext.create`, `xtype`,
  `dataIndex`, `renderTo`, `Ext.Msg`, `grid.getStore()`, `getSelectionModel()`, `store.sync()`) or the
  user mentions an Ext JS / Sencha background.
metadata:
  tags: bryntum, extjs, sencha, migration, scheduler, gantt, grid, schedulerpro
---

# Ext JS → Bryntum migration

Turns an Ext JS app (Classic or Modern) into a Bryntum app with no Ext left. For an incremental path, the Ext
components are replaced first and the Ext shell is removed afterwards.

**Version.** The target is Bryntum **8.0**: tiered packages (`@bryntum/<product>-<tier>`, tier = `community`,
`core` or `enterprise`) on public npm. Pick the branch with the version check in the core `bryntum` skill. Every
code block in this skill and its references is 8.0 code, except where a line says "7.x". For an app that stays on 7.x
(or a Bryntum-in-Ext screen already on 7.x, see below) read `references/v7.md` too: it lists what differs.

Not migrating, just writing Bryntum code with Ext JS habits? Read `references/idioms.md`: Bryntum shares Ext's
vocabulary (`Store`, `items`, `renderer`, `type`) but several look-alike calls fail silently.

Load the core `bryntum` skill and the target's framework skill as well. Add `bryntum-crud` for a real backend,
`bryntum-editor` when the app customizes the event/task editor, and `bryntum-features` to find the built-in feature
for an Ext plugin or hand-written behavior.

Read only the references the app needs. If this skill isn't installed locally, fetch them from
`https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-extjs-migration/references/<file>`.

| File | Use it for |
|---|---|
| `references/api-mapping.md` | Ext class / config / plugin / column / method → Bryntum, what has no equivalent, and outdated advice to ignore |
| `references/idioms.md` | Line-by-line porting: store / record / tree / component-lookup methods, and the Ext near-misses that fail silently |
| `references/data-migration.md` | Models, stores, field mapping vs data conversion, CrudManager, Gantt data changes |
| `references/patterns.md` | Idiom rewrites: app shell, classes, toolbars, dialogs, editors, renderers, locales, undo/redo, framework targets |
| `references/styling.md` | Theme choice, dropping Ext chrome, selector renames, color tokens |
| `references/products/<product>.md` | Product-specific gotchas and support level: `grid`, `scheduler`, `schedulerpro` (Scheduler at the Enterprise tier on 8.0), `gantt`, `calendar`, `taskboard` |
| `references/templates.md` | Inventory checklist, `MIGRATION_PLAN.md` / `MIGRATION_REPORT.md` formats, verification checklist |
| `references/v7.md` | Only for 7.x targets: package names, trial/licensed install, imports, CSS, Scheduler Pro, known 7.3.7 bugs |

## Principles

- **Only use names you can confirm.** Old Ext → Bryntum material is full of plausible names that don't exist in 7.x or 8.x.
  Look names up in this order: `references/` (checked against the 8.0.0 packages; items tagged 7.3.7 were not
  rechecked on 8.x), then `mcp__bryntum__search_bryntum_docs` (pass `product` and the installed `version`), then the package source in `node_modules/@bryntum/` (its `*.d.ts` typings are the quickest check when the docs
  search tool isn't available). If none of these confirms a name, treat
  the item as **unmapped**: leave a `// MIGRATION:` comment where it was and list it in the report.
- **Report every loss.** Anything removed, changed or unmapped goes in the report with a reason. Count the key records
  before and after, and include both counts in the report.
- **Write to a new folder or branch**, not over the original app. The team needs the original to compare against.
- **Write idiomatic Bryntum, not an Ext emulation.** Use features, keyed `items` + `widgetMap`, and a shared
  store/project. Don't build a compatibility layer for `Ext.*` APIs.
- **Check with the user before a big job.** Show the plan's mapped/changed/unmapped counts before writing code for
  anything larger than one screen. Stop and ask if more than a third of the items are unmapped, or if the app needs a
  backend you can't run.

## Before starting

Ask the user about anything they haven't already told you, and record the answers in `MIGRATION_PLAN.md`:

- **Tier and license.** The tiers are Community, **Core** and Enterprise (Core was named Business until 2026-11-10; the published
  8.0.0 packages still use `-business` and `tier : 'business'`, so confirm names with `npm view`). Every `@bryntum/*` package in the app is the same tier at the same exact version. Use the tier
  the user names or holds a license for ("free" / "open source" means Community). Otherwise use **Enterprise**, say you
  picked it, and say it runs as a 14-day trial until `bryntum.licenseKey` is set (Community has no key). Don't choose the
  lowest tier by feature fit, but check every Ext feature you port against the chosen tier (`references/api-mapping.md`
  §0): a feature above the tier is skipped and a dialog names the tier it needs, so leave it out and list it as a gap.
  Never invent a key: ask once, or leave a `// TODO: bryntum.licenseKey = '...'` placeholder.
  Tier needs in practice (details in `references/api-mapping.md` §0): an Ext Gantt needs **Core** at least
  (constraints, calendars, dependencies and `taskEdit` are Core; Community has a lighter engine) and **Enterprise**
  for baselines, critical paths, `projectEdit`, Rollups, MS Project export and print. (Gantt `summary` and `pdfExport` are Core.) An
  Ext Scheduler that relied on the scheduling engine (dependencies that move events, constraints, effort, calendars: the
  former Scheduler Pro) needs **Enterprise**. Calendar and TaskBoard keep the Core data layer at Enterprise.
  Per-feature tiers: `static tier` in the installed package is authoritative; the `feature-availability.md` guides can list a higher tier.
- **Install.** Public npm, no login, no `-trial` package: `npm install --save-exact @bryntum/<product>-<tier>@8.x.y`
  (or the framework wrapper, e.g. `@bryntum/gantt-react-enterprise`). `npm view @bryntum/<product>-<tier> version`
  gives the version. Untiered `@bryntum/<product>` on public npm is an empty placeholder: never install it. A
  `@bryntum:registry=https://npm.bryntum.com` line in `~/.npmrc` or the project makes every tiered package 404: add a
  project `.npmrc` with `@bryntum:registry=https://registry.npmjs.org/` (Yarn v2+ `npmScopes.bryntum.npmRegistryServer`,
  Yarn v1 `"@bryntum:registry"` in `.yarnrc`) instead of deleting the user-level mapping, which other 7.x projects
  still need. If the package can't be installed, do the work that doesn't need it and mark verification as blocked.
- **Target stack**: vanilla + Vite (closest to Ext's config-object style), React, Angular or Vue, in TS or JS.
- **Big-bang or incremental.** Incremental means Bryntum widgets run inside the Ext shell for a while, with the
  Bryntum build loaded next to Ext (`references/patterns.md` §11). 8.0 ships **no prebuilt UMD bundle**: load ES
  modules (`<script type="module">`, as the suite's Ext demos do) or build your own UMD bundle with webpack
  (`gettingstarted/build-umd.md`, which still publishes the `bryntum.<product>` global). Confirm the team accepts that
  interim state.
- **Backend**: keep the server's JSON shape and map the fields, or change the data to Bryntum's shape.
- **Scope**: the whole app or named screens. For a large app, migrate one screen end to end as a pilot.

## Classify the source (per screen — apps are often mixed)

| Type | How to tell | What changes |
|---|---|---|
| **A. Legacy Ext Scheduler / Gantt / Calendar / TaskBoard** | `Sch.*`, `Gnt.*`, `Cal.*`, `Kanban.*`, `ptype` plugins, PascalCase data (`StartDate`) | Everything: shell, configs, plugins, columns, renderers, **data** |
| **B. Bryntum embedded in Ext** | `bryntum.<product>.*` globals or wrappers, an Ext panel around a Bryntum widget, camelCase data | Only the Ext shell. Copy the Bryntum configs inside and re-check them against the target version's defaults. The data usually copies as is. If that Bryntum code is 7.x or older, the screen also needs the 7.x → 8.x hop (package renames, tiers, imports, Scheduler Pro → Scheduler): do it with `bryntum-migrate` |
| **C. Plain Ext components** | `Ext.grid.Panel`, `Ext.tree.Panel`, no Bryntum | `Grid` / `TreeGrid` (`references/products/grid.md`). Other Ext UI maps to the framework or to Bryntum widgets |

Flag these for a human instead of migrating them by guesswork: Ext Charts, Pivot, `Ext.calendar` (the open-source Ext
component; Bryntum Calendar is possible, but mark it limited — Bryntum's own commercial `Cal.*` is type A), Ext Direct, routing, and overrides of private Ext/Sch/Gnt internals.

## Converting

Build the inventory and plan from the checklist in `references/templates.md`. For scaffolding (install, CSS, sizing,
Vite `optimizeDeps`), follow the core skill. Points specific to migrations:

- Remove every Ext script, stylesheet and build file. Rebuild the styling for Bryntum 8 instead of porting the Ext
  chrome (`references/styling.md`).
- Choose **one** data strategy per app, field mapping or conversion (`references/data-migration.md`). Gantt calendars,
  `TaskType` and baselines can only be converted.
- Set `barMargin`, `rowHeight`, `viewPreset` and `startDate`/`endDate` explicitly. Their defaults differ from Ext.
- Don't copy Ext pixel widths as is: Bryntum cells are padded and themes are roomier, so text clips and fixed-size
  content squeezes (`references/styling.md`, "Sizes from Ext").
- Ext's header menu differs from `headerMenu`; match it through `items` (`references/products/grid.md`).
- Convert plugins to `features` only when the mapping is confirmed.
- Some Bryntum features are on by default that the Ext app may not have had: `cellEdit`, `cellMenu`, `headerMenu`,
  `columnReorder`, `group`, `rowCopyPaste` (Grid, Scheduler); `eventEdit`, `eventMenu`, `scheduleMenu`,
  `eventTooltip`, `eventCopyPaste`, `enableDeleteKey` (Scheduler); `taskCopyPaste` (Gantt). Turn off each one
  the source lacked, or keep it and list it as a behavior change. `cellEdit` takes over the row double-click that Ext
  apps often used to open an edit window. `cellMenu`'s "Remove row" and the Delete key both skip any app-level
  delete confirmation. Two defaults that change Ext behavior without a feature flag: multi-row selection
  (`selectionMode.multiSelect`) and Ctrl-click multi-column sort (`sort : { multiSort }`); Ext grids select one row
  and sort one column.
- Import each class from the package that owns it: an 8.0 package exports its own classes only (`Store`,
  `DateHelper`, `Popup`, `Toolbar`, `LocaleHelper` from `@bryntum/core-<tier>`, `Column` from `@bryntum/grid-<tier>`).
  Features of another product register only when that package is imported (`import '@bryntum/grid-<tier>'` for
  `cellTooltip` in a Scheduler).
- In a Vite target set `build.cssTarget` (core skill, "Using with Vite"), or the built app loses its theme colors while
  `npm run dev` looks right. Leave `optimizeDeps` alone.
- Ext apps often replaced the event editor with a custom dialog. Customize the built-in `eventEdit`/`taskEdit`
  instead. Other dialogs become a `Popup` subclass, or the host framework's dialog when the app has its own component
  system.
- Controllers and ViewModel `bind` become explicit handlers (vanilla) or framework state.
- Only add localization, undo/redo or RTL if the source app had them.

## Verifying

Use the checklist in `references/templates.md`. Three lessons from past migrations:

- Run the dev server as well as `npm run build`. Vite's dev server enforces file-serving rules that the build doesn't,
  so icons and fonts can 404 only in dev.
- Drive the UI the way a user would, with Playwright clicks, right-clicks and typing. Helpers like
  `showContextMenuFor()` skip the real code path and prove nothing. For a vanilla target, expose the root widget on
  `window`. For a framework target, use the wrapper's `instance`.
- Check every claim before writing it. "Fits", "no longer clipped" or "matches Ext" needs a screenshot or the same
  input sequence run on the original. Proving that a feature works doesn't prove it behaves like Ext.

Then write `MIGRATION_REPORT.md` (`references/templates.md`). Follow the core skill's hand-off, and suggest Bryntum
features the Ext app didn't have.

## Traps

- The official Ext → Bryntum guides and old blog posts predate 7.x in places. `references/api-mapping.md` §9 lists the
  corrections.
- 7.3.7 bug (recheck on 8.x): `items : { someBuiltIn : true }` on a menu or editor blanks the built-in item. Leave kept
  built-ins out of `items`, or override only the properties you change.
- Setting `eventEdit`/`taskEdit` to `false` also removes the `beforeEventEdit` hook. For a custom dialog, keep the
  feature enabled and return `false` from the hook (`bryntum-editor`).
- 7.3.7 bug (recheck on 8.x): a function `field` in the store's `groupers` config throws (`store.group(fn)` works). Group on a
  calculated field instead.
- A mapping for one product doesn't carry over to another (Scheduler `eventRenderer` vs Gantt `taskRenderer`).
- Ext-shaped calls that look right fail quietly in Bryntum: `store.first()` (a getter), `store.find('name', x)`
  (takes a predicate; use `findRecord`), `node.insertChild(index, child)` (reversed, takes a sibling), `down()` (use
  `widgetMap`), and, on 7.x, setting an undeclared record field (never saved; on 8.0 a column's `field` is added to the model automatically, others still are not). Check ported code against
  `references/idioms.md`.
- When switching tier, version or registry, move `package-lock.json` and `node_modules/` aside first. The lockfile
  silently keeps the old package source (look for `npm.bryntum.com` in `resolved` URLs).
- Verify the license state in the browser: no badge on Community or with a valid key; a Trial badge on Core /
  Enterprise without a key. Report either, and a "requires the Core/Enterprise tier" dialog, in the report.
