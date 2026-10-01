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

Turns an Ext JS app (Classic or Modern) into a Bryntum 7 app with no Ext left. For an incremental path, the Ext
components are replaced first and the Ext shell is removed afterwards.

Load the core `bryntum` skill and the target's framework skill as well. Add `bryntum-crud` for a real backend and
`bryntum-editor` when the app customizes the event/task editor.

Read only the references the app needs. If this skill isn't installed locally, fetch them from
`https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-extjs-migration/references/<file>`.

| File | Use it for |
|---|---|
| `references/api-mapping.md` | Ext class / config / plugin / column / method → Bryntum, what has no equivalent, and outdated advice to ignore |
| `references/data-migration.md` | Models, stores, field mapping vs data conversion, CrudManager, Gantt data changes |
| `references/patterns.md` | Idiom rewrites: app shell, classes, toolbars, dialogs, editors, renderers, locales, undo/redo, framework targets |
| `references/styling.md` | Theme choice, dropping Ext chrome, selector renames, color tokens |
| `references/products/<product>.md` | Product-specific gotchas and support level: `grid`, `scheduler`, `schedulerpro`, `gantt` |
| `references/templates.md` | Inventory checklist, `MIGRATION_PLAN.md` / `MIGRATION_REPORT.md` formats, verification checklist |

## Principles

- **Only use names you can confirm.** Old Ext → Bryntum material is full of plausible names that don't exist in 7.x.
  Look names up in this order: `references/` (verified against 7.3.7), then `mcp__bryntum__search_bryntum_docs` for
  the installed version, then the package source in `node_modules/@bryntum/`. If none of these confirms a name, treat
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

- **Trial or licensed package.** Trial: `npm install @bryntum/{product}@npm:@bryntum/{product}-trial` (watermarked).
  Licensed: the `npm.bryntum.com` registry or a local build. `@bryntum/{product}` on public npm is a placeholder. If
  licensed access fails, do the work that doesn't need the package and mark verification as blocked. Don't switch to
  the trial without asking.
- **Target stack**: vanilla + Vite (closest to Ext's config-object style), React, Angular or Vue, in TS or JS.
- **Big-bang or incremental.** Incremental means Bryntum widgets run inside the Ext shell for a while, with the
  UMD/module build loaded next to Ext. Confirm the team accepts that interim state.
- **Backend**: keep the server's JSON shape and map the fields, or change the data to Bryntum's shape.
- **Scope**: the whole app or named screens. For a large app, migrate one screen end to end as a pilot.

## Classify the source (per screen — apps are often mixed)

| Type | How to tell | What changes |
|---|---|---|
| **A. Legacy Ext Scheduler / Gantt** | `Sch.*`, `Gnt.*`, `ptype` plugins, PascalCase data (`StartDate`) | Everything: shell, configs, plugins, columns, renderers, **data** |
| **B. Bryntum embedded in Ext** | `bryntum.<product>.*` globals or wrappers, an Ext panel around a Bryntum widget, camelCase data | Only the Ext shell. Copy the Bryntum configs inside and re-check them against 7.x defaults. The data usually copies as is |
| **C. Plain Ext components** | `Ext.grid.Panel`, `Ext.tree.Panel`, no Bryntum | `Grid` / `TreeGrid` (`references/products/grid.md`). Other Ext UI maps to the framework or to Bryntum widgets |

Flag these for a human instead of migrating them by guesswork: Ext Charts, Pivot, `Ext.calendar` (Bryntum Calendar is
possible, but mark it limited), Ext Direct, routing, and overrides of private Ext/Sch/Gnt internals.

## Converting

Build the inventory and plan from the checklist in `references/templates.md`. For scaffolding (install, CSS, sizing,
Vite `optimizeDeps`), follow the core skill. Points specific to migrations:

- Remove every Ext script, stylesheet and build file. Rebuild the styling for Bryntum 7 instead of porting the Ext
  chrome (`references/styling.md`).
- Choose **one** data strategy per app, field mapping or conversion (`references/data-migration.md`). Gantt calendars,
  `TaskType` and baselines can only be converted.
- Set `barMargin`, `rowHeight`, `viewPreset` and `startDate`/`endDate` explicitly. Their defaults differ from Ext.
- Convert plugins to `features` only when the mapping is confirmed.
- Some Bryntum features are on by default that the Ext app may not have had: `cellEdit`, `cellMenu`, `headerMenu`
  (Grid); `eventEdit`, `eventMenu`, `scheduleMenu`, `eventTooltip`, `enableDeleteKey` (Scheduler). Turn off each one
  the source lacked, or keep it and list it as a behavior change. `cellEdit` takes over the row double-click that Ext
  apps often used to open an edit window. `cellMenu`'s "Remove row" and the Delete key both skip any app-level
  delete confirmation.
- Ext apps often replaced the event editor with a custom dialog. Customize the built-in `eventEdit`/`taskEdit`
  instead. Other dialogs become a `Popup` subclass, or the host framework's dialog when the app has its own component
  system.
- Controllers and ViewModel `bind` become explicit handlers (vanilla) or framework state.
- Only add localization, undo/redo or RTL if the source app had them.

## Verifying

Use the checklist in `references/templates.md`. Two lessons from past migrations:

- Run the dev server as well as `npm run build`. Vite's dev server enforces file-serving rules that the build doesn't,
  so icons and fonts can 404 only in dev.
- Drive the UI the way a user would, with Playwright clicks, right-clicks and typing. Helpers like
  `showContextMenuFor()` skip the real code path and prove nothing. For a vanilla target, expose the root widget on
  `window`. For a framework target, use the wrapper's `instance`.

Then write `MIGRATION_REPORT.md` (`references/templates.md`). Follow the core skill's hand-off, and suggest Bryntum
features the Ext app didn't have.

## Traps

- The official Ext → Bryntum guides and old blog posts predate 7.x in places. `references/api-mapping.md` §9 lists the
  corrections.
- 7.3.7 bug: `items : { someBuiltIn : true }` on a menu or editor blanks the built-in item. Leave kept
  built-ins out of `items`, or override only the properties you change.
- Setting `eventEdit`/`taskEdit` to `false` also removes the `beforeEventEdit` hook. For a custom dialog, keep the
  feature enabled and return `false` from the hook (`bryntum-editor`).
- 7.3.7 bug: a function `field` in the store's `groupers` config throws (`store.group(fn)` works). Group on a
  calculated field instead.
- A mapping for one product doesn't carry over to another (Scheduler `eventRenderer` vs Gantt `taskRenderer`).
- When switching between trial and licensed packages, move `package-lock.json` and `node_modules/` aside first. The
  lockfile silently keeps the old package source.
