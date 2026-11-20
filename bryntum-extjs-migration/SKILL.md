---
name: bryntum-extjs-migration
description: >
  Migrate an Ext JS application to Bryntum 8 (or 7.x) — legacy Ext Scheduler / Gantt (`Sch.*`, `Gnt.*`),
  Bryntum components wrapped in an Ext JS shell, or plain Ext grids and tree grids
  (`Ext.grid.Panel`, `Ext.tree.Panel`), Ext calendars (`Extensible.calendar.*`) and hand-built Ext boards
  (kanban) — targeting vanilla JS, React, Angular, or Vue. Use alongside
  the `bryntum` skill. Trigger on phrases like "migrate from Ext JS", "port our Ext app", "replace
  Sencha", "Ext JS end of life", "Sch.panel", "Gnt.panel", "Ext.grid.Panel to Bryntum", or any
  Ext JS codebase being moved to Bryntum. Also use it whenever Ext JS habits appear in Bryntum code
  (`store.first()`, `component.down()`, `store.get(id)`, `Ext.getCmp`, `Ext.create`, `xtype`,
  `dataIndex`, `renderTo`, `Ext.Msg`, `Ext.Ajax`, `Ext.define`, `grid.getStore()`, `getSelectionModel()`, `store.sync()`) or the
  user mentions an Ext JS / Sencha background.
metadata:
  tags: bryntum, extjs, sencha, migration, scheduler, gantt, grid, schedulerpro, calendar, taskboard
---

# Ext JS → Bryntum migration

Turns an Ext JS app (Classic or Modern) into a Bryntum app with no Ext left. For an incremental path, the Ext
components are replaced first and the Ext shell is removed afterwards.

**Version.** The target is Bryntum **8.0**: tiered packages (`@bryntum/<product>-<tier>`, tier = `community`,
`core` or `enterprise`) on public npm. Pick the branch with the version check in the base `bryntum` skill. Every
code block in this skill and its references is 8.0 code, except where a line says "7.x". For an app that stays on 7.x
(or a Bryntum-in-Ext screen already on 7.x, see below) read `references/v7.md` too: it lists what differs.

Bryntum's Ext Scheduler (`Sch.*`) and Ext Gantt (`Gnt.*`) add-ons are discontinued: today's products are standalone ES modules with no Ext dependency and no shared code, so forum threads and answers under `Sch.*`/`Gnt.*` don't apply.

**Stale base skill.** This skill is 8.0. If the base `bryntum` skill you loaded has no "Tiers" section (it names
`@bryntum/gantt-trial`, `@bryntum/gantt`, `@bryntum/{product}/svalbard-light.css` or `optimizeDeps.include`), it is the
7.x copy: this skill wins, and the current text is at
`https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md`. Following the stale copy installs
packages that don't exist.

Not migrating, just writing Bryntum code with Ext JS habits? Read `references/idioms.md`: Bryntum shares Ext's
vocabulary (`Store`, `items`, `renderer`, `type`) but several look-alike calls fail silently.

Load the base `bryntum` skill and the target's framework skill as well. Add `bryntum-crud` for a real backend,
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
| `references/products/<product>.md` | Product-specific gotchas and support level: `grid`, `scheduler`, `scheduler-enterprise` (Scheduler at the Enterprise tier on 8.0), `gantt`, `calendar`, `taskboard` |
| `references/templates.md` | Inventory checklist, `MIGRATION_PLAN.md` / `MIGRATION_REPORT.md` formats, verification checklist |
| `references/v7.md` | Only for 7.x targets: package names, trial/licensed install, imports, CSS, Scheduler Pro, known 7.3.7 bugs |

## Principles

- **Only use names you can confirm.** Old Ext → Bryntum material is full of plausible names that don't exist in 7.x or 8.x.
  Look names up in this order: `references/` (checked against the 8.0.0 packages; 7.x-only behavior is in
  `references/v7.md`), then `mcp__bryntum__search_bryntum_docs` (pass `product` and the installed `version`), then the package source in `node_modules/@bryntum/` (its `*.d.ts` typings are the quickest check when the docs
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

- **Tier and license.** Follow the base `bryntum` skill's "Tiers", "License key" and "Installing packages" sections for the
  tier names (Community, Core, Enterprise; install `@bryntum/<product>-core`, never the deprecated `-business`), the
  `@bryntum/core-core` package name, the exact-version rule, the trial and the `.npmrc` registry trap. A `-business` app is
  pinned to 8.0.0 and renames through `bryntum-migrate`. Use the tier the user names or holds a license for ("free" /
  "open source" means the Community tier). Otherwise use the **Enterprise** tier, say you picked it, and say it runs as a
  14-day trial until `bryntum.licenseKey` is set (the Community tier has no key). Don't choose the lowest tier by feature
  fit, but check every Ext feature you port against the chosen tier (`references/api-mapping.md` §0): a feature above the
  tier is skipped and a dialog names the tier it needs, so leave it out and list it as a gap.
  Never invent a key: ask once, or leave a `// TODO: bryntum.licenseKey = '...'` placeholder.
  Tier needs in practice (details in `references/api-mapping.md` §0): an Ext Gantt needs the Core tier at least
  (constraints, calendars, dependencies and `taskEdit` are Core tier; the Community tier has a lighter engine) and the Enterprise tier
  for baselines, critical paths, `projectEdit`, Rollups, `summary`, `pdfExport`, MS Project export and print. An
  Ext Scheduler that relied on the scheduling engine (dependencies that move events, constraints, effort, calendars: the
  former Scheduler Pro) needs the Enterprise tier. Calendar and TaskBoard keep the Core tier data layer at the Enterprise tier.
  A Scheduler with no engine feature and no Enterprise-only feature ran identically at the Core tier in two test ports (drag,
  resize, drag-create, editor, menus, `filterBar`, `workingTime`, `presets`, `timeRanges`) and the Core bundle is about 25%
  smaller. That is an observation, not the default: the default stays the Enterprise tier, and the tier choice is the user's
  (name Core as an option in the plan).
- **Install.** Follow the base skill's "Installing packages": public npm, no login, no `-trial` package,
  `npm install --save-exact @bryntum/<product>-<tier>@8.x.y` (or the framework wrapper, e.g.
  `@bryntum/gantt-react-enterprise`; wrapper names are `-react-`, `-vue-`, `-angular-`, there is no `-vue-3-`). Add the
  wrapper of every product on the page (a `BryntumGrid` next to a Scheduler needs `@bryntum/grid-react-<tier>` too). A 404
  from `npm view @bryntum/<product>-<tier> version` means an
  `npm.bryntum.com` mapping is active: add a project `.npmrc` with `@bryntum:registry=https://registry.npmjs.org/` (Yarn
  v2+ `npmScopes.bryntum.npmRegistryServer`, Yarn v1 `"@bryntum:registry"` in `.yarnrc`) instead of falling back to
  `-business` or deleting the user-level mapping, which other 7.x projects still need. npm reads that `.npmrc` only in
  a folder with a `package.json` (run `npm init -y` first), and `--registry` doesn't override the scoped mapping: outside a
  project use `npm view <pkg> version --@bryntum:registry=https://registry.npmjs.org/`. If the package can't be
  installed, do the work that doesn't need it and mark verification as blocked.
- **Target stack**: vanilla + Vite (closest to Ext's config-object style), React, Angular or Vue, in TS or JS.
- **Big-bang or incremental.** Incremental means Bryntum widgets run inside the Ext shell for a while, with the
  Bryntum build loaded next to Ext (`references/patterns.md` §11). 8.0 ships **no prebuilt UMD bundle**: load ES
  modules (`<script type="module">`) or build your own UMD bundle with webpack
  (`gettingstarted/build-umd.md`, which still publishes the `bryntum.<product>` global). Confirm the team accepts that
  interim state.
- **Backend**: keep the server's JSON shape and map the fields, or change the data to Bryntum's shape.
- **Scope**: the whole app or named screens. For a large app, migrate one screen end to end as a pilot.

Then read the references your app needs **before writing code**: agents in test rounds that skipped them hit errors the
references already answer. Always read `references/api-mapping.md` (§0 tiers and the product's own table) and
`references/products/<product>.md`; add `references/patterns.md` §6 for any app with forms or dialogs,
`references/data-migration.md` for any backend, and `references/idioms.md` for ported handler code.

## Classify the source (per screen — apps are often mixed)

| Type | How to tell | What changes |
|---|---|---|
| **A. Legacy Ext Scheduler / Gantt / Calendar / TaskBoard** | `Sch.*`, `Gnt.*`, `Cal.*`, `Kanban.*`, `ptype` plugins, PascalCase data (`StartDate`) | Everything: shell, configs, plugins, columns, renderers, **data** |
| **B. Bryntum embedded in Ext** | `bryntum.<product>.*` globals or wrappers, an Ext panel around a Bryntum widget, camelCase data | Only the Ext shell. Copy the Bryntum configs inside and re-check them against the target version's defaults. The data usually copies as is. If that Bryntum code is 7.x or older, the screen also needs the 7.x → 8.x hop (package renames, tiers, imports, Scheduler Pro → Scheduler): do it with `bryntum-migrate` |
| **C. Plain Ext components** | `Ext.grid.Panel`, `Ext.tree.Panel`, `Ext.view.View` / dataviews, `Ext.dd.*`, no Bryntum | `Grid` / `TreeGrid` (`references/products/grid.md`), or a `TaskBoard` for a hand-built board (`references/products/taskboard.md`). Other Ext UI maps to the framework or to Bryntum widgets |

- **Judge by what the code does, not by the repo or class name.** Real cases: a repo called "Kanban" that was a marketing
  page (Ext containers with `html` plus one dataview) needs no Bryntum widget: port it to plain HTML/CSS/JS and report
  "no Bryntum widget needed". A repo with no `Kanban.*` that was an hbox of dataview columns, one per status, is a type C
  board and becomes a `TaskBoard`.
- **The `bryntum-scheduler` / `bryntum-gantt` Ext packages (`Sch.*`, `Gnt.*`) are type A**, even though Bryntum made
  them. Type B is the standalone `@bryntum/*` / `bryntum.*` build inside an Ext shell.
- **The open-source Ext calendar is Extensible** (`Extensible.calendar.*`, `xtype : 'extensible.calendarpanel'`, Ext 3/4).
  It is not `Cal.*` and not type A: see `references/products/calendar.md`, "Source: Extensible".
- **Large app:** migrate one screen end to end as a pilot and list the other screens in the plan as a table. When the
  original can't run (it needs a backend or SDK you don't have), build a mock from the models and proxies, verify the
  Bryntum side only, and say what was not compared.

Flag these for a human instead of migrating them by guesswork: Ext Charts, Pivot, Ext 6 `Ext.calendar.*` (the commercial
Ext Calendar package; Bryntum Calendar is possible, but mark it limited), routing, and overrides of private Ext/Sch/Gnt
internals. Ext Direct (`directFn`, `Ext.direct.*`) is not Bryntum's to speak: `references/api-mapping.md` §8 has the
recipe (keep the server contract, write a thin RPC client).

## Converting

Build the inventory and plan from the checklist in `references/templates.md`. For scaffolding (install, CSS, sizing,
Vite `optimizeDeps`), follow the base skill. Points specific to migrations:

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
  `eventTooltip`, `eventCopyPaste`, `enableDeleteKey`, `stickyEvents`, `eventFilter`, `nonWorkingTime`,
  `timeAxisHeaderMenu` (Scheduler); `taskCopyPaste`, `projectLines` (an orange "Project
  start" line), `percentBar` (its drag handle swallows task drags on short bars), `taskEdit`, `taskMenu`, `dependencies`
  (terminals on every bar) (Gantt); `taskEdit`, `taskMenu`, `columnHeaderMenu`, `columnToolbars` (a "+" under each column)
  (TaskBoard). **At the Enterprise tier a Scheduler also has** `taskEdit` (the default editor: `eventEdit : false` alone
  still opens it on double-click and on drag-create, so set `taskEdit : false` too), `dependencies`, `dependencyMenu`,
  `eventSegments`, `eventSegmentDrag`, `eventSegmentResize`. List what is on in the running app with
  `Object.keys(widget.features).filter(k => !widget.features[k].disabled)`. Turn off each one the source lacked, or keep
  it and list it as a behavior change.
- **Features inside a raw widget config** (an entry in `items`, a nested `type : 'grid'`) use `features : { x : false }`. The
  `xFeature` prop form exists only on the wrapper element; inside `items` it is silently ignored and the feature stays on
  (seen in an Angular round: `cellEditFeature : false` in `items` left cell editing on).
- **Wrappers take features as props, not as a `features` object.** The `features : { x : cfg }` blocks in this skill are
  vanilla. In React, Vue and Angular write one prop per feature: `eventEditFeature={...}` (React, Vue),
  `[eventEditFeature]="..."` (Angular); `false` disables it. A `features` prop is a TypeScript error in Vue and Angular
  and **silently ignored in React** (it compiles, there is no warning, and the feature keeps its default).
- `cellEdit` takes over the row double-click that Ext
  apps often used to open an edit window. `cellMenu`'s "Remove row" and the Delete key both skip any app-level
  delete confirmation. Two defaults that change Ext behavior without a feature flag: multi-row selection
  (`selectionMode.multiSelect`) and Ctrl-click multi-column sort (`sort : { multiSort }`); Ext grids select one row
  and sort one column.
- Import each class from the package that owns it: an 8.0 package exports its own classes only (`Store`,
  `DateHelper`, `Popup`, `Toolbar`, `LocaleHelper` from `@bryntum/core-<tier>`, `Column` from `@bryntum/grid-<tier>`).
  Features of another product register only when that package is imported (`import '@bryntum/grid-<tier>'` for
  `cellTooltip` in a Scheduler).
- In a Vite target set `build.cssTarget` (base skill, "Using with Vite"), or the built app loses its theme colors while
  `npm run dev` looks right. It matters for the suffix-less family sheets (`light-dark()`); a fixed `-light`/`-dark`
  sheet is unaffected (tested), but setting it anyway is harmless. Leave `optimizeDeps` alone.
- Ext apps often replaced the event editor with a custom dialog. Customize the built-in `eventEdit`/`taskEdit`
  instead. Other dialogs become a `Popup` subclass, or the host framework's dialog when the app has its own component
  system.
- Controllers and ViewModel `bind` become explicit handlers (vanilla) or framework state.
- Only add localization, undo/redo or RTL if the source app had them.

## Verifying

Use the checklist in `references/templates.md`. Lessons from past migrations:

- Run the dev server as well as `npm run build`. Vite's dev server enforces file-serving rules that the build doesn't,
  so icons and fonts can 404 only in dev.
- Drive the UI the way a user would, with Playwright clicks, right-clicks and typing. Helpers like
  `showContextMenuFor()` skip the real code path and prove nothing. For a vanilla target, expose the root widget on
  `window`. For a framework target, use the wrapper's `instance`.
- Run the original, compare navigation and counts, and don't port an apparent bug silently (one app's menu opened the
  wrong screen in the original): port the intent and list it as a behavior change.
- Assert after a real drop, not just that it renders. Silent failures found this way: a chained store that kept the dragged
  row, a `dataSource` mapping that hid the source key, a sorted store that swallowed a reorder.
- Test the failure paths, not only the happy path. For every write (add, edit, delete, drag) make the server fail once and
  confirm the screen does what the original did: error shown, edit reverted, row restored. Silent failures found this
  way: a failed delete with no message, an edit that stayed on screen after the server rejected it.
- Check every claim before writing it. "Fits", "no longer clipped" or "matches Ext" needs a screenshot or the same
  input sequence run on the original. Proving that a feature works doesn't prove it behaves like Ext.

Finish in this order: (1) clean the folder: no build artefacts (`tsconfig.tsbuildinfo`, `dist/`), no scripts that read
files outside the project, no leftover Ext or IDE folders (`.idea`, `.vscode`) you can remove, pinned dependency versions;
(2) write `MIGRATION_REPORT.md` (`references/templates.md`: never skip it, even when a README exists); (3) suggest Bryntum
features the Ext app didn't have.

## Traps

- The official Ext → Bryntum guides and old blog posts predate 7.x in places. `references/api-mapping.md` §9 lists the
  corrections.
- 8.0.0, until bryntum/bryntum#14231 is fixed: `items : { deleteEvent : true }` on a menu renders the Delete item with
  empty text. Leave kept built-ins out of `items`, or override only the properties you change.
- Setting `eventEdit`/`taskEdit` to `false` also removes the `beforeEventEdit` hook. For a custom dialog of your own,
  keep the feature enabled and return `false` from the hook (`bryntum-editor`). To only customize the built-in editor, configure
  `eventEdit`, and at the Enterprise tier also set `taskEdit : false` (`references/products/scheduler.md`).
- 8.0.0, until bryntum/bryntum#14230 is fixed: a function `field` in `groupers`, `addGrouper()` or `group({ field : fn })`
  throws `field.indexOf is not a function`, and `store.group(fn)` silently groups nothing. Group on a calculated field.
- `grid.disabled = true` neither greys the grid nor stops a row click selecting (8.0.1, bryntum/bryntum#14249; `api-mapping.md` §7): use
  `'inert'` to lock a list while a form is dirty.
- A Scheduler-owned feature used in a Gantt (`pan`, `timeRanges` for `showTodayLine`) throws `Invalid type name` until the
  Scheduler package is imported (`api-mapping.md`, Gantt table).
- A mapping for one product doesn't carry over to another (Scheduler `eventRenderer` vs Gantt `taskRenderer`).
- Ext-shaped calls that look right fail quietly in Bryntum: `store.first()` (a getter), `store.find('name', x)`
  (takes a predicate; use `findRecord`), `node.insertChild(index, child)` (reversed, takes a sibling), `down()` (use
  `widgetMap`), and, on 7.x, setting an undeclared record field (never saved; on 8.0 a column's `field` is added to the model automatically, others still are not). Check ported code against
  `references/idioms.md`.
- When switching tier, version or registry, move `package-lock.json` and `node_modules/` aside first. The lockfile
  silently keeps the old package source (look for `npm.bryntum.com` in `resolved` URLs).
- Verify the license state in the browser: no badge at the Community tier or with a valid key; a Trial badge at the Core or
  Enterprise tier without a key. Report either, and a "requires the Core/Enterprise tier" dialog, in the report. A second product component (e.g. the Gantt's `ResourceGrid`)
  can log its own trial message; with a key, check that it covers every product used.
