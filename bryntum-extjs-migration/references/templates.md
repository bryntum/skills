# Inventory, plan, report and verification

## Inventory checklist

Read every file in scope and give each item its own row in the plan:

- app shell: `Ext.application`, controllers, `Viewport`, `requires`
- every `Ext.define`: views, models, stores, plugins, overrides
- root widget configs, every `plugins` entry, every column (`xtype`, `dataIndex`, `filter`, editor)
- renderers and templates: `eventRenderer`, `taskBodyTemplate`, `getRowClass`, `*Tpl`
- toolbars, buttons and their handlers; dialogs and forms
- stores, models and proxies, plus the data they load; product structures (calendars, baselines, assignments,
  dependencies)
- ViewModel bindings, localization, undo/redo, custom CSS

Also record the toolkit (Classic / Modern), Ext version, build (Sencha Cmd or other) and backend proxies.

A workable conversion order: root widget + data hub → configs → plugins → columns → renderers → toolbars/dialogs →
models → app logic → extras → styling.

## MIGRATION_PLAN.md

```markdown
# Migration plan: <app or screen> → Bryntum <Product> <version> (<target stack>)

- **Package:** trial | licensed (registry) | licensed (local build) | licensed (unverified — verification blocked)
- **Target:** <vanilla + Vite | React | Angular | Vue 3>, <TypeScript | JavaScript>
- **Approach:** big-bang | incremental (phase 1: Bryntum inside Ext shell; phase 2: shell removed)
- **Source:** <path>, Ext JS <version> <Classic|Modern>, type <A|B|C> per screen
- **Data strategy:** field mapping | data conversion — <why>
- **Summary:** <n> items: <n> mapped, <n> changed, <n> unmapped

| Ext item | Where (file:line) | Bryntum target | Source of mapping | Status |
|---|---|---|---|---|
| `Sch.panel.SchedulerGrid` | app/view/Main.js:12 | `Scheduler` | api-mapping §1 (G-S) | mapped |
| `plugins : ['scheduler_pan']` | app/view/Main.js:40 | `features.pan` | api-mapping §5 (SRC) | mapped |
| `Ext.chart.CartesianChart` | app/view/Stats.js:5 | — | api-mapping §8 | unmapped |

## Unmapped (needs a decision)
- <item> — <why> — <suggested options, if any>
```

"Source of mapping" is a reference section + tag, an MCP/docs lookup (name the page), or the package source file.

## MIGRATION_REPORT.md

```markdown
# Migration report: <app or screen> → Bryntum <Product> <version> (<target stack>)

- **Source:** <path> (<n> JS files, <n> lines)
- **Target:** <package + version>, <stack> (<n> files, <n> lines)
- **Package:** <as in the plan>
- **Status:** ✅ verified | ⚠️ verified with open items | ⛔ migration prepared, verification blocked (<reason>)

## File-by-file
| Ext file | What it did | Result |

## Config mapping
| Ext config / plugin / column | Bryntum | Source of mapping |

## Data
- Strategy: <field mapping | conversion> — script: <path, if any>
- Counts before → after: resources 10 → 10, events 42 → 42, assignments …, dependencies …
- Structural changes: <calendars → intervals, baselines → array, TaskType removed, …>
- Backend changes required: <none | list>

## Unmapped / dropped
| Item | Why | Alternative used (if any) |
Include every dropped CSS rule with its bucket (Ext chrome / theme mimicry / dead selector / restates default).

## Behavior differences
- <what users will notice, and why>

## Verification
| Check | Result |

## Next steps
- Real Bryntum features the Ext app didn't have that fit this app (with docs links)
```

## Verification checklist

Check both the `npm run build` output and the dev server, using headless Chromium via Playwright.

Load and data:

- [ ] build passes; dev server starts; FontAwesome icons and fonts load in dev (no 404s)
- [ ] 0 console errors and 0 warnings on load (including no "sized by its predefined minHeight" warning)
- [ ] store/project record counts equal the counts recorded in the report
- [ ] the component renders data — rows/bars visible, not an empty timeline

Structure:

- [ ] every planned column exists, in order, with the planned header text
- [ ] every planned feature exists on the root widget with the expected enabled/disabled state
- [ ] product structures render when the data included them (dependencies, baselines, time ranges, calendars'
  non-working time)

Interaction:

- [ ] each toolbar button: click it and assert its effect
- [ ] context menus: every visible item has text (a blank item means a built-in config was replaced)
- [ ] event/task editor opens where the source had one, shows custom fields, saves to the record (check the saved
  values, not just the count), and Cancel discards; a newly drag-created record is removed on cancel
- [ ] each other dialog: correct title and prefilled values; required fields block Save; Save and Enter both work;
  Cancel/close discards
- [ ] drag/resize/create where the source allowed it; validators reject what the source rejected
- [ ] backend: a create/update/delete round-trips (sync request shape accepted, phantom IDs replaced)
- [ ] test targets are on screen — clicks on off-screen elements silently do nothing

Look:

- [ ] no Ext stylesheet or `.x-*` selector remains
- [ ] theme and font are as planned; no leftover Ext colors on Bryntum internals
- [ ] screenshots taken after popups/tooltips close and after show animations finish (a popup or toast captured
  mid-animation looks semi-transparent); compared with the original; differences listed in the report
- [ ] original won't run because the Ext/Sch vendor files are missing (common: Sch/Gnt are commercial)? In a scratch
  copy, load the Ext JS GPL build from cdnjs (`extjs` 6.2.0 / 6.0.x / 4.2.1: `ext-all-debug.js` plus the theme's
  `theme-*.js` and `theme-*-all.css`) and stub the `Sch.*`/`Gnt.*` views with placeholder panels. This covers the
  non-scheduler screens only

- [ ] at the original's window width: no clipped header or cell text, fixed-size content keeps its shape, no
  toolbar items pushed into an overflow menu (`styling.md`, "Sizes from Ext")

Report:

- [ ] each "fits"/"fixed"/"verified"/"matches Ext" claim names the check or screenshot behind it; a "matches Ext"
  claim ran the same input sequence on the original, not just proved the feature works

After about 5 fix-and-recheck rounds, stop and report exactly what still fails.
