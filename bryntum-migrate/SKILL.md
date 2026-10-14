---
name: bryntum-migrate
description: >
  Upgrade an existing Bryntum app (Gantt, Scheduler Pro, Scheduler, Calendar, Grid, TaskBoard)
  to a newer Bryntum version — 6.x → 7.x, 7.x → 7.y, 7.x → 8.x, 8.x → 8.y (including the 8.0
  tier package rename `@bryntum/<product>` → `@bryntum/<product>-<tier>` and Scheduler Pro →
  `@bryntum/scheduler-enterprise`). Gathers the upgrade guides and changelogs for every release
  crossed, across the product and the products it inherits from, checks them against the app's
  code, writes a migration plan for approval, then applies and verifies it. Use for "upgrade /
  update / migrate Bryntum to <version>", "bump @bryntum/*", "what changed between 6.x and 7.x",
  "my Gantt/Scheduler broke after updating", or the error "CSS version X doesn't match bundle
  version Y". Not for migrating from another vendor (DHTMLX, Syncfusion, FullCalendar, …) or
  first-time installation (use the `bryntum` skill).
metadata:
  tags: bryntum, migrate, upgrade, update, changelog, breaking-changes, codemod, versions, schedulerpro, tier
---

Load the `bryntum` skill and the app's framework skill alongside this one (fallback: `https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/<skill>/SKILL.md`). For an 8.x target, their package names and CSS paths are 7.x ones: `v7-to-v8.md` wins.

For API lookups use `mcp__bryntum__search_bryntum_docs` with the **target** `version`. Without the MCP, read `<docs host>/products/<product>/docs-llm/api/<Product>/<path>/<Class>.md` (e.g. `.../docs-llm/api/Core/helper/TimeZoneHelper.md`; `docs-llm/llms.txt` lists them all), or grep the packed target package.

## References

Load these when you reach the step that needs them. They sit in `references/` next to this file, or at `https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-migrate/references/<file>`.

| File | When |
|------|------|
| `sources.md` | Gathering material: the migration index, fallback docs URLs, sibling products, what to read closely |
| `v6-to-v7.md` | Range crosses 7.0.0: kebab-case CSS, new themes, FontAwesome, Button colours, moved classes, `*Data` props, the CSS codemod |
| `v7-to-v8.md` | Range crosses 8.0.0: tier package names, imports from owning packages, CSS paths, Scheduler Pro, Vue 2/Angular/UMD blockers, checklist |
| `plan-template.md` | Writing the plan file |
| `worked-example.md` | A real Gantt 6.0.3 → 7.2.1 hop, to calibrate scope |

## Gotchas

- **All `@bryntum/*` product and wrapper packages must be on the same exact version.** A mismatch throws `CSS version X doesn't match bundle version Y` at runtime; one that already exists is the first plan item. 7.x `@bryntum/*-lib` helpers (gone in 8.0) have their own numbers and come in as dependencies. `@bryntum/demo-resources` follows the product version from 7.0.0, so bump it too.
- **Take the installed version from `node_modules/@bryntum/<pkg>/package.json`** (or the lockfile), not the semver range in `package.json`. For zip installs, read `build/package.json`.
- **Breaking changes arrive through the products the app inherits from.** A Gantt app breaks on Grid and Scheduler changes too, and Core/Chart changes appear in their own changelogs (and, from 8.0.0, a Core upgrade guide). Read every sibling's material, not just the top-level product's.
- **8.0 renames every package** (`@bryntum/<product>-<tier>`). The upgrade guides say so, but other docs and READMEs can still show 7.x names. Take names and paths from `v7-to-v8.md`, the upgrade guide and the packed target.
- **A green build doesn't mean the migration is done.** Bryntum deprecations only show at runtime, in the browser console, and some never log at all (Later.js schedule parsing, deprecated in 7.3.0, runs silently in 7.3.7; `TimeZoneHelper` and `smoothZoom`, deprecated in 8.0.0, don't log either). List the deprecations named in the guides and changelogs yourself.
- **Custom CSS can stop matching without being renamed.** The rename tables only cover class renames, not classes that moved to another element. Check custom selectors against the running app (see `v6-to-v7.md`).

## Workflow

### 1. Detect

- **Product and framework** from `@bryntum/*` dependencies. Strip `-trial`, `-thin`, the 8.x tier suffix (`-community`, `-business`, `-enterprise`) and the wrapper suffix (`-react`, `-angular`, `-vue-3`, and `-vue` on 8.x). On 7.x, `-vue` without `-3` is the Vue 2 wrapper and `-angular-view` the View Engine wrapper. If several products are installed (for example Gantt + Grid), run the workflow for the most specific one, because its sibling list covers the rest.
- **Zip install**: there's no `@bryntum/*` dependency, but a local `build/package.json` exists. The zip ships `docs/`, `changelog.md` and a `migrate.js` codemod.
- **Target version**: use the one the user gave, otherwise the `latest` tag (`npm view <package> dist-tags`; the `versions` list ends with `-nightly` builds, so don't take its last entry). From 8.0 the package is `@bryntum/<product>-<tier>`: the untiered name has no 8.x versions and its `latest` stays on 7.x. **Check the target exists** with `npm view <target package>@<version> version` before planning. If it doesn't, stop and tell the user. This skill doesn't downgrade; a target equal to the installed version means nothing to do; for a prerelease target, warn that it has no guides yet and confirm first.
- **Framework wrapper compatibility**: the wrapper's README is the real constraint (`peerDependencies` is empty). For a target ≥ 8.0.0, check the blockers in `v7-to-v8.md` (Vue 2, Angular < 15, UMD) and ask for the licence tier.
- Tell the user what you found in one block: product, package rename, framework, installed → target, sibling products, package manager, blockers. Then continue.

### 2. Gather

Follow `sources.md`: try the migration index first. Add the hop-specific notes from `v6-to-v7.md` / `v7-to-v8.md`. Turn what you read into candidate items: `{ version, product, source URL + heading, kind, old, new, grep terms }`. The kind is one of breaking, behaviour change, deprecation or cosmetic. Grep terms are identifiers that would appear in customer code: config, method, event and class names, CSS selectors, import paths.

### 3. Plan, then stop for approval

Grep the customer's source (excluding `node_modules` and build output) for each item's terms and mark it *applies* (list the files), *not used*, or *unsure*. Use *unsure* for dynamic access, string-built keys, spread configs and generic terms. Write the plan using `plan-template.md`.

- **Grep more than the source.** Data files (`.json`: `*Data` keys, `iconCls`, `eventColor`, Later.js schedule strings), HTML templates (`.html`, `.ejs`) and build config (`angular.json` `styles` / `assets`, `vite.config.*` `optimizeDeps` / `onLog`, `package.json` scripts such as a `postinstall` that copies Bryntum CSS) often hold the hits. Skip gitignored copies of Bryntum files (theme CSS copied into `public/`) and your own scratch folder.
- **New defaults have no grep terms.** Guides also announce defaults that changed ("DependencyMenu is now enabled by default", "Group member rows are now indented"). They affect every app that uses the feature, including features that are on by default (Grid's `group`, Calendar's `eventCopyPaste`), so list each one under *Default behaviour changes* with its opt-out instead of marking it *not used*. The widget element shows the active features as `b-<feature>` classes: record them in the baseline and diff them after the upgrade, because not every new default is documented.

Two cheap checks before asking for approval, neither of which touches the project:

- **Baseline the current app.** Run it and capture screenshots, plus the computed styles of custom-styled elements, after each app action you will want to check later (the old version is gone once you install). You need this to compare against after the upgrade, and it also shows which custom rules already didn't match. Write the script's selectors to survive the v7 kebab-case renames (`v6-to-v7.md`). Pin the browser time zone (`page.emulateTimezone`) for an app that shows times.
- **Inspect the target package.** From the app root (so its `.npmrc` registry applies), `npm pack <target package>@<version> --pack-destination <scratch>` for the product, its wrapper and `core-<tier>`, and read the product package's `README.md` / `AGENTS.md` first: they give the current import and CSS paths. Then check CSS file names, the FontAwesome path, the exports (`entry.js`, `*.d.ts`, a wrapper's `src/index.ts`) and whether the configs and classes the plan relies on exist. Check every path the app reaches into a `@bryntum/*` package by name: SCSS `@import`s, deep imports, `postinstall` copy maps (`npm pack --dry-run` lists the files without downloading).

Don't edit anything until the user approves the plan, and that includes `package.json`. Ask with `AskUserQuestion` if it's available, otherwise list the questions numbered and ask the user to reply, or to edit the plan and reply "approve". Re-read the plan afterwards and apply what it says, not your memory of it. Write the user's answers into the plan under each question, and rewrite an action's New code when an answer changes it. Leave unanswered *unsure* items unapplied.

### 4. Apply

- **Zip install:** the user downloads the target zip and replaces `build/` (and `docs/`), then re-runs this skill so the versions are detected again.
- Edit `package.json` first, then run one install with the project's package manager (`npm uninstall` + `npm install` reorders the file). Rename packages (`v7-to-v8.md`), set every `@bryntum/*` dependency to the exact target version with no `^` or `~`, and update `postinstall` scripts that copy Bryntum files, since the install runs them; clear every one of their old destinations first. Then confirm every `@bryntum/*` package in `node_modules` reports the target version and no old name is left.
- For a 6 → 7 hop, run the CSS codemod if it's available (the index's `tools.migrate6to7` or a zip's `migrate.js`), dry-run first. Otherwise handle the selector renames as plan actions (`v6-to-v7.md`).
- Apply the plan's actions in order and tick each one off in the plan file. Skip items marked *not applicable*.
- Move CSS and theme imports to the target layout: `v7-to-v8.md` for 8.x, the `bryntum` skill's CSS setup section for 7.x.
- Restart any running dev server after installing, with the bundler's cache cleared (`vite --force`, delete `.angular/cache`), so it re-optimises `@bryntum/*`.

### 5. Verify

- Run the project's build/typecheck and tests. Fix errors the migration caused, and report unrelated ones separately. If there are no tests, script the app's own UI actions (toolbar buttons, editors, menus, sorting, grouping, the editor on double-click) in a headless browser.
- Compare against the baseline screenshots and computed styles. Check fixed-width columns for truncation, and check custom renderer output.
- Grep for every identifier and package name the plan removed or renamed, as whole words (`grep -w`: a plain `SchedulerPro` also matches inside `BryntumSchedulerProps` otherwise). There should be no hits.
- If a dev server can run, check the browser console. Bryntum's `VersionHelper.deprecate('<removal version>', msg)` logs `Deprecation warning: You are using a deprecated API which will change in v<version>. <message>`; once that version ships, the same call throws `Deprecated API use. <message>`. Add each warning to the plan as a follow-up. Missing structural, theme or FontAwesome CSS is loaded from a CDN with a warning containing `CSS issues detected`, so a correctly styled page doesn't prove the CSS imports are right; that warning must be absent too, as must requests to `cdn.jsdelivr.net`. Warnings only fire on code paths that run, so a page-load check alone is partial.
- Stop every dev server you started, and check its port afterwards. A server started through `npx` can keep listening after its shell is stopped; start it as `node node_modules/<tool>/bin/<tool>.js` instead, or kill the process holding the port (Windows: `Get-NetTCPConnection -State Listen | ? LocalPort -eq <port> | % { Stop-Process -Id $_.OwningProcess }`; Unix: `kill $(lsof -ti :<port>)`).
- Report the applied items, the remaining manual and *unsure* items, and the build/test status. Remind the user to check the console after the first real run.
