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

Detect products by namespace: `Sch.*` → Scheduler (engine use → Scheduler Enterprise), `Gnt.*` → Gantt, `Cal.*` or
`Extensible.calendar.*` → Calendar, `Kanban.*` → TaskBoard; a hand-built board (`Ext.view.View` columns per status) is a
TaskBoard too, and a page of containers with `html` needs no Bryntum widget (`SKILL.md`, "Classify the source"). A screen may
mix them (shared resource store); report each separately.

Also record the toolkit (Classic / Modern), Ext version, build (Sencha Cmd or other) and backend proxies.

A workable conversion order: root widget + data hub → configs → plugins → columns → renderers → toolbars/dialogs →
models → app logic → extras → styling.

## MIGRATION_PLAN.md

```markdown
# Migration plan: <app or screen> → Bryntum <Product> <version> (<target stack>)

- **Bryntum:** 8.x | 7.x (`<exact version>`), tier community | core | enterprise (chosen by: user | license | default), exact package names (`@bryntum/<product>-community`, `-core` for the Core tier, `-enterprise`)
- **License:** key registered | trial (Core or Enterprise tier, 14 days) | none needed (Community tier) | install blocked — verification blocked
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
- **Bryntum / tier / license:** <as in the plan>
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

- [ ] every `@bryntum/*` dependency is one tier and one exact version; no `npm.bryntum.com` in `.npmrc` or the lockfile; `npm view @bryntum/<product>-<tier> version` prints a version
- [ ] build passes; `npm run preview` (the production build) shows theme colors; dev server starts; FontAwesome icons and fonts load in dev (no 404s)
- [ ] 0 console errors and 0 warnings on load (including no "sized by its predefined minHeight" warning)
- [ ] store/project record counts equal the counts recorded in the report
- [ ] Gantt: dump `start`/`end` from the original and from `project.taskStore` and diff them; list every task that moved (the
  engine recalculates dependency-violating source dates: `data-migration.md`)
- [ ] navigation and menus on the original: don't port an apparent bug silently (a menu that opened the wrong screen in the
  original); port the intent and report it
- [ ] the component renders data — rows/bars visible, not an empty timeline
- [ ] license state as planned: no badge (Community tier or valid key), or a Trial badge and console message (no key; report it); no "requires the Core/Enterprise tier" dialog — if one shows, drop the feature or report the tier it needs

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
- [ ] backend: a create/update/delete round-trips (sync request shape accepted, phantom IDs replaced); for a failed write,
  the original's error path (dialog, revert or reload) is reproduced. Force a failure **for each of add, edit and delete**
  (delete is the one that fails silently) and look at the screen afterwards
- [ ] after each real drop, assert the effect (the row left the source list, the order field changed, the server got the
  call): "it renders" missed three silent failures
- [ ] test targets are on screen — clicks on off-screen elements silently do nothing

Look:

- [ ] no Ext stylesheet or `.x-*` selector remains
- [ ] theme and font are as planned; no leftover Ext colors on Bryntum internals
- [ ] screenshots taken after popups/tooltips close and after show animations finish (a popup or toast captured
  mid-animation looks semi-transparent); compared with the original; differences listed in the report
- [ ] original won't run because the Ext/Sch vendor files are missing (common: Sch/Gnt are commercial)? In a scratch
  copy, load the Ext JS GPL build from cdnjs (`extjs` 6.2.0 / 6.0.x: `ext-all-debug.js` plus the theme's `theme-*.js` and
  `theme-*-all.css`) and stub the `Sch.*`/`Gnt.*` views with placeholder panels. This covers the non-scheduler screens only.
  Things that cost hours:
  - cdnjs' `theme-triton-all.css` is a two-line `@import` stub: fetch `theme-triton-all_1.css`, `_2.css`, the `fonts/` and
    `images/` folders next to them, or the page renders unstyled
  - an app whose `ext` is a dead symlink names the SDK version in its loader (`ext-6.0.0`): serve cdnjs 6.2.0 under that path
  - Ext 4.2.x GPL: `cdn.sencha.com/ext/gpl/4.2.1/` serves the individual files over **http** (the zip URL is a 404): route the
    requests from an https test page. Another checkout in the workspace may already hold an Ext 4 copy
  - a Sencha Cmd app: the dev `bootstrap.js` can fail (`Sch is not defined`: `Main.js` uses `Sch.util.Date` before the loader
    has `gantt/`), and `build/production/` can be older than the source. Check the build against the source, then use a scratch
    page that loads `ext-all.js`, the vendor bundle (`gnt-all.js` / `sch-all.js`), `app/Application.js`, the views and
    `app.js` plus the app and theme CSS
  - a PHP backend: `php -S localhost:<port> -d display_errors=0` (PHP 8.4 deprecation notices break the JSON); MySQL-bound
    code needs a Node mock of the wire contract
  - Playwright against the original: drag-create and hover may never fire in headless Chromium; if two attempts fail, say
    "matches Ext" was not claimed for them
  - a trial bundle shows a watermark but no 2026 expiry problem was seen

- [ ] at the original's window width: no clipped header or cell text, fixed-size content keeps its shape, no
  toolbar items pushed into an overflow menu (`styling.md`, "Sizes from Ext")

Report:

- [ ] every sentence under Behavior changes and Verification is backed by a line of code you read or a run you did: delete
  any claim you did not check ("only one edit window", "no cell editing", "parent durations match", "server errors are
  shown" were each false in a test run)
- [ ] before you write "no equivalent", "not possible" or "unmapped": search the installed packages and the docs for the Bryntum name
  (`grep -rn "<ExtConfig>\|<likely name>" node_modules/@bryntum/*/lib`, the docs' `llms.txt`). All 3 E4 runs reported the histogram scale as
  impossible; it is `scalePoints`, and the skill's own row said the same until it was fixed
- [ ] hygiene: server files that emit Ext (`ExtDirect.php` returning Ext.Direct JS) are Ext code: replace or delete them (all 3 E3 runs
  kept it); no request to a third party at runtime (Google Fonts: ship the font or use the app's own, 1 E5 run loaded it from outside);
  no `tsconfig.tsbuildinfo`/`dist`, no scripts reading `../original`, dependency versions pinned, backup
  folders you refer to are really delivered
- [ ] each "fits"/"fixed"/"verified"/"matches Ext" claim names the check or screenshot behind it; a "matches Ext"
  claim ran the same input sequence on the original, not just proved the feature works

After about 5 fix-and-recheck rounds, stop and report exactly what still fails.
