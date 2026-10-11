# 7.x → 8.x hop

Applies when `installed < 8.0.0 <= target`.

## Scheduler Pro becomes Scheduler

Scheduler Pro is merged into Scheduler in 8.0 as its Enterprise tier (`tier : 'enterprise'`; tiers are `'community' | 'business' | 'enterprise'`). The `@bryntum/schedulerpro` package line ends at 7.x. Every `@bryntum/schedulerpro*` package (`-react`, `-angular`, `-vue-3`, `-thin`, `-trial`) becomes the matching `@bryntum/scheduler*` package. `SchedulerPro` / `SchedulerProBase` and the `schedulerpro` / `schedulerprobase` widget types survive as deprecated aliases until 9.0.0, so old code runs but warns.

Derive the product from the installed package, then map it for the target. The 7.x part of the range reads Scheduler Pro guides; the 8.x part reads Scheduler guides. Record the rename in the plan header.

When applying, do the rename before the version bump:

- `package.json`: `@bryntum/schedulerpro<suffix>` → `@bryntum/scheduler<suffix>` (same suffix, same trial alias shape).
- Imports `from '@bryntum/schedulerpro…'` → `from '@bryntum/scheduler…'`.
- `new SchedulerPro({` → `new Scheduler({ tier : 'enterprise',`; `type : 'schedulerpro'` → `type : 'scheduler', tier : 'enterprise'`.
- CSS `schedulerpro.css` → `scheduler.css`.
- Wrapper component and selector names exactly as the wrapper's 8.0.0 guide spells them.

A leftover `@bryntum/schedulerpro` next to `@bryntum/scheduler` in `node_modules` loads two bundles, so make sure the install removes it.

## Environment blockers

Check these during detection; each hit is a mandatory plan item.

- Angular below 15 (`@angular/core`) — 8.0 requires 15+. The `-angular-view` (Angular ≤ 11 View Engine) package is no longer published.
- Vue 2 wrapper (`@bryntum/<product>-vue`, no `-3`) — removed in 8.0.0; 7.3.x is the last Vue 2 line. Reaching 8.x requires a Vue 3 migration first.
- UMD bundle (`*.umd.js` import or `<script src=".../<product>.umd.js">`) — no longer shipped; ES modules only.

## Headline changes checklist

The 8.0.0 upgrade guides hold the details and the Old/New code — work from them, not from this table. Use it as a checklist the plan answers row by row with *applies* / *not used*.

| Headline change (8.0.0) | Grep the customer code for |
|-------------------------|----------------------------|
| Scheduler Pro merged into Scheduler (above); prefer `isEnterpriseTier` over `isSchedulerPro`; `*Enterprise` model/store classes are resolved from the tier — use tier-neutral `ProjectModel`, `EventModel`, … | `@bryntum/schedulerpro`, `SchedulerPro`, `schedulerpro`, `isSchedulerPro`, `ProjectModelPro`, `*Enterprise` |
| UMD bundle no longer shipped — build your own with webpack if needed | `.umd.js`, `bryntum.<product>` global |
| Vue 2 wrapper removed | `@bryntum/<product>-vue"`, `vue@2` |
| Angular 15+ required; `-angular-view` gone | `@angular/core` version, `-angular-view` |
| `GridFeatureManager` removed — features register via the `Factoryable` pattern, `defaultEnabled` may be keyed by tier | `GridFeatureManager`, `registerFeature` |
| Project is now a CrudManager; standalone `CrudManager` class and `crudManager` config deprecated (removed 9.0.0); `scheduler.crudManager` returns the project | `new CrudManager`, `crudManager :`, `CrudManager load response` |
| Color API emits predefined `b-` names (`b-red`) instead of hex; swatches use `.b-color-<name>`; `DomHelper.resolveColorValue` / `createColorStyle` for literals | `eventColor`, `ColorField`, `ColorPicker`, `ColorColumn`, hex comparisons on color fields |
| New Temporal-based time zone implementation is the default | `timeZone`, `TimeZoneHelper` |
| Vendored `later.js` removed | `Engine/vendor/later`, `later.` |
| Themes also ship as family sheets (`svalbard.css` renders light/dark from `color-scheme`) — optional, no migration required | theme `<link>` / `@import`, `setTheme` |
| Per-product extras named in the guides (e.g. `amPm` preset → `sixHoursAndDay`, Grid column virtualization, Calendar DayView sticky content, Gantt `toggleParentTasksOnClick` default) | whatever the guide heading names |
