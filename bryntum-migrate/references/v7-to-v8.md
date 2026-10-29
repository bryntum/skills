# 7.x → 8.x hop

Applies when `installed < 8.0.0 <= target`. Each 8.0.0 upgrade guide opens with the package rename, but other docs
(integration and quick-start guides, the wrapper packages' READMEs) can still show 7.x names (`@bryntum/<product>`,
`-vue-3`, `-thin`). Take package names, import paths and CSS paths from this file, the upgrade guide, the product
package's `README.md` / `AGENTS.md`, and the wrapper's exports (`src/index.ts`, `public-api.d.ts`).

## Registry prerequisite

8.x installs from public npm (`https://registry.npmjs.org/`), unless the user supplies a local or other target registry.
Before checking package availability or packing the target, pass `--@bryntum:registry=<target-registry>` to `npm view`
and `npm pack`. This overrides an old scoped mapping to `npm.bryntum.com` or `npm-us.bryntum.com`; `--registry` alone
does not. These command-line overrides leave the customer's configuration unchanged while the plan is prepared.

Include project and CI registry changes in the plan: override or remove the old Bryntum mapping in `.npmrc` or Yarn's
`.yarnrc.yml` so the eventual install uses the selected target registry. Check inherited user configuration too; avoid
changing shared user settings that other 7.x apps still need. Apply approved project and CI changes before installing.
Read the full `docs-llm/guide/<Product>/migration/migrate-from-v7-to-v8.md` guide linked from the 8.0.0 upgrade guide
for registry migration and license-key setup.

## Every package is renamed: `@bryntum/<product>-<tier>`

8.0 ships each product in three cumulative tiers, `community < business < enterprise`, and the tier is part of every
package name. Query the tiered name for 8.x: untiered names on the private registry stay on 7.x, while public npm can
have untiered placeholder packages without product code.

| 7.x package | 8.0 package |
|-------------|-------------|
| `@bryntum/<product>`, `@bryntum/<product>-thin` | `@bryntum/<product>-<tier>` |
| `@bryntum/<product>-react`, `-angular` (and `-thin` variants) | `@bryntum/<product>-react-<tier>`, `-angular-<tier>` |
| `@bryntum/<product>-vue-3` (and `-vue-3-thin`) | `@bryntum/<product>-vue-<tier>`, `vue` with no `-3` |
| `@bryntum/schedulerpro*` | `@bryntum/scheduler-enterprise`, `@bryntum/scheduler-<framework>-enterprise` |
| `@bryntum/<product>-lib` helpers | gone, the tier packages depend on `@bryntum/core-<tier>`, `grid-<tier>`, ... |
| `-trial`, `-angular-view`, `-vue` (Vue 2) | not published for 8.0, see the blockers below |
| `@bryntum/demo-resources` | unchanged name, version follows the product |

`<product>` is `grid`, `scheduler`, `gantt`, `calendar`, `taskboard`, `core`, `chart` or `engine`. Chart has an
`-enterprise` package only.

**The tier is the customer's licence, so ask.** First check which tiers the customer's registry has
(`npm view @bryntum/<product>-{community,business,enterprise} version`), then put the question under *Unsure* in the
plan, with the lowest tier the app needs. For Scheduler Pro, only confirm Enterprise.

- Check what the app uses against the tier of each class in the packed target: `grep -rl "static tier = '" lib/`
  lists the classes that need more than Community (for example Grid's `Group` feature is Business). A lower-tier
  package doesn't contain higher-tier classes: an imported class fails the build, a feature enabled by config fails
  at runtime (`Invalid type name`).
- Read the product's `guides/data/tiers.md` (docs-llm) for what each tier adds. Feature tiers are in `static tier`
  (Calendar also has `CalendarFeature.featureTiers`), widget and config tiers only in `@tier` source comments.
- **Calendar and TaskBoard keep the basic data layer at every tier**, as in 7.x: their models and project resolve to
  the Business classes even in the Enterprise package. Calendar's Enterprise tier adds the DayAgendaView, ResourceView,
  DayResourceView, MonthAgendaView, MonthGrid, YearPlanner and MultiWeekView modes; the scheduling engine is opt-in,
  through an explicit `ProjectModelEnterprise` project. Base custom models on the plain names Calendar and TaskBoard
  export themselves (`EventModel`, `ResourceModel`, ...), not on Scheduler's.

## Imports: the product package no longer re-exports what it is built on

8.0 packages are "thin": `@bryntum/calendar-enterprise` exports Calendar's own classes only. A 7.x app that imports
`DateHelper`, `StringHelper`, `Toast`, `EventModel`, ... from its product package must import each class from the
package that owns it (`@bryntum/core-<tier>`, `@bryntum/scheduler-<tier>`, ...). The same goes for wrappers:
`BryntumDemoHeader` / `<bryntum-demo-header>`, `BryntumButton` and other Core widgets come from
`@bryntum/core-<framework>-<tier>` (in Angular, the template doesn't compile without `BryntumCoreModule`). Find the
owner by grepping the packed packages' `entry.js` (product) or `src/index.ts` / `public-api.d.ts` (wrappers).

**Load every product package the app's features come from.** A thin product package registers only the inherited
features its own code imports. A Gantt app that enables `timeRanges` (a Scheduler feature) with only the Gantt
package loaded builds fine and then throws `Invalid type name "timeRanges" passed to GanttFeature factory`. Import
the inherited packages too: in Angular `BryntumGridModule` + `BryntumSchedulerModule` next to `BryntumGanttModule`
(and `BryntumCoreModule` for `<bryntum-demo-header>`), as the wrapper README's Quick Start does; elsewhere an
`import '@bryntum/scheduler-<tier>'`. A green build proves nothing here, so check the console for errors.

The owning packages arrive as dependencies of the product package. Add each one the app imports from to
`package.json` at the same version: strict package managers (pnpm, Yarn PnP) only resolve declared dependencies.

## CSS and FontAwesome

- FontAwesome ships only in `@bryntum/core-<tier>/fontawesome/` (no `fontawesome/` in the product package).
- Each product package has its structural sheet in two forms: `<product>.css` (includes Core, Grid, ... for a
  single-product app) and `<product>.thin.css` (that product only, for apps that combine products). Themes ship in
  every package; the README imports them from `@bryntum/core-<tier>`.
- No `.min.css` files. Rewrite every copy of Bryntum files that names the old package, `*.min.css` or
  `<product>/fontawesome/**`: a `postinstall` copy map, Angular `angular.json` `assets` globs, webpack copy plugins.
  Copy themes from the product package (identical to core's) and FontAwesome from `core-<tier>`.
- `@bryntum/demo-resources` 8.0's demo header lists theme families and loads family sheets (`svalbard.css`), with
  a light/dark toggle.

```css
@import "@bryntum/core-enterprise/fontawesome/css/fontawesome.css";
@import "@bryntum/core-enterprise/fontawesome/css/solid.css";
@import "@bryntum/gantt-enterprise/gantt.css";
@import "@bryntum/core-enterprise/svalbard-light.css";
```

## Build

8.0 packages ship ES module sources (`entry.js` + `lib/**`) instead of one `<product>.module.js`. Rollup reports
`CIRCULAR_DEPENDENCY` warnings inside `core-<tier>` and `engine-<tier>`. Apps started from a 7.x Bryntum Vite demo
turn every Rollup warning into an error in `build.rollupOptions.onLog`, so the build fails until that hook skips
them, as the 8.0 demos do:

```js
onLog(level, log, handler) {
    if (log.code === 'CIRCULAR_DEPENDENCY') {
        return;
    }
    ...
}
```

In Vite, remove the old `@bryntum/*` names from `optimizeDeps.include`. Add the tiered names of every `@bryntum/*` package the app imports from only if the console reports "The Bryntum <Product> bundle was loaded multiple times by the application".

## Scheduler Pro becomes Scheduler

Map `@bryntum/schedulerpro*` to `@bryntum/scheduler-*-enterprise` (table above). The 7.x part of the range reads
the Scheduler Pro material; the 8.x part reads Scheduler's (`/products/scheduler/...`; there is no Scheduler Pro index
for 8.x). In code, per the Scheduler 8.0.0 guide ("Scheduler Pro is merged into Scheduler"):

- `new SchedulerPro({` → `new Scheduler({ tier : 'enterprise',`; `type : 'schedulerpro'` → `type : 'scheduler',
  tier : 'enterprise'`.
- Wrappers: `BryntumSchedulerPro` → `BryntumScheduler`, `BryntumSchedulerProProps` → `BryntumSchedulerProps`,
  `<bryntum-scheduler-pro>` → `<bryntum-scheduler>`, plus `tier : 'enterprise'` in the config.
- `schedulerpro.css` → `scheduler.css`.

The old classes and wrapper components are deprecated aliases until 9.0.0: they run, and log a deprecation warning.
Make sure the install leaves no `@bryntum/schedulerpro*` in `node_modules`.

## Blockers

Check these during detection. Each hit is a mandatory plan item, and the first two stop the upgrade until the
customer decides.

- **Vue 2** (`@bryntum/<product>-vue`, no `-3`): no 8.0 wrapper. 7.3.x is the last Vue 2 line. Ask: stay on 7.3.x, or
  migrate to Vue 3 first (Vue CLI below 4.5 can't build Vue 3, so the toolchain changes too). The product's Vue
  integration guide (`docs-llm/guide/<Product>/integration/vue/guide.md`) shows the Vue 3 wrapper.
- **Angular below 15** (`@angular/core`), or `-angular-view`: 8.0 needs Angular 15+.
- **UMD bundle** (`*.umd.js`, `bryntum.<product>` global): no longer shipped. The `gettingstarted/build-umd.md`
  guide shows how to build one with webpack.

## Checklist

The 8.0.0 upgrade guides (Core, Grid, Scheduler and each product) hold the Old/New code; answer each row in the plan.
Also read the 8.0.0 changelogs' `[BREAKING]` / `[DEPRECATED]` entries: some changes are only there.

| Change | Grep the customer code for |
|--------|----------------------------|
| Project is a CrudManager; `crudManager` config and standalone `CrudManager` deprecated | `crudManager`, `new CrudManager` |
| Colors emit `b-` names (`b-red`); helpers that build `var(--b-color-${color})` break | `eventColor`, `color :`, `--b-color-`, `ColorField` |
| Temporal time zones are the default (see below); `TimeZoneHelper` deprecated | `timeZone`, `TimeZoneHelper` |
| `smoothZoom` deprecated, never logs (smooth zoom is the default); `zoomFactor` 2 → 1.5 | `smoothZoom`, `zoomFactor` |
| `amPm` preset → `sixHoursAndDay`; vendored `later.js` removed | `amPm`, `later.` |
| `GridFeatureManager` removed (features use `Factoryable`) | `GridFeatureManager`, `registerFeature` |
| Columns virtualized, `autoAddField` on, group rows indented, overscroll off, `detectEventBarLayoutIssues` on (new defaults) | always list under *Default behaviour changes* || Themes also ship as family sheets (`svalbard.css`), optional | theme `<link>` / `@import` |
| Product extras named in each guide (Gantt `toggleParentTasksOnClick`, Calendar DayView sticky content and YearView cells, renderer argument order, ...) | whatever the heading names |

### Time zone code

In Temporal mode a view's `timeZone` only changes what the view displays. Dates you read from and pass to the view
are real moments in time, and the view's `timeZone` no longer sets the project's (`project.timeZone` stays separate,
so working-time calendars stay in their own zone). App code that converted dates for the view now shifts them twice:

```js
// 7.x
gantt.startDate = TimeZoneHelper.toTimeZone(currentDay, gantt.timeZone);
// 8.0
gantt.startDate = currentDay;
```

Pin the browser zone in the baseline (`page.emulateTimezone`) and compare header labels and bar positions in a few
zones before and after, not stored dates: 7.x stored zone-shifted values, 8.0 stores the real time. `useLegacyTimeZoneMode : true` on the project restores the 7.x behaviour (deprecated).
