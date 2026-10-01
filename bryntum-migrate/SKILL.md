---
name: bryntum-migrate
description: >
  Upgrade an existing Bryntum app (Gantt, Scheduler Pro, Scheduler, Calendar, Grid, TaskBoard)
  to a newer Bryntum version — 6.x → 7.x, 7.x → 7.y, 7.x → 8.x, 8.x → 8.y (including Scheduler Pro →
  Scheduler `tier : 'enterprise'`). Gathers the upgrade guides and changelogs for every release
  crossed, across the product and the products it inherits from, checks them against the app's
  code, writes a migration plan for approval, then applies and verifies it. Use for "upgrade /
  update / migrate Bryntum to <version>", "bump @bryntum/*", "what changed between 6.x and 7.x",
  "my Gantt/Scheduler broke after updating", or the error "CSS version X doesn't match bundle
  version Y". Not for migrating from another vendor (DHTMLX, Syncfusion, FullCalendar, …) or
  first-time installation (use the `bryntum` skill).
metadata:
  tags: bryntum, migrate, upgrade, update, changelog, breaking-changes, codemod, versions, schedulerpro, tier
---

Load the `bryntum` skill and the app's framework skill alongside this one (fallback: `https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/<skill>/SKILL.md`).

For API lookups use `mcp__bryntum__search_bryntum_docs` with the **target** `version` — it doesn't index upgrade guides or changelogs. If the MCP isn't available, ask the user to add it: `claude mcp add --transport http bryntum https://mcp.bryntum.com`.

## References

Load these when you reach the step that needs them. They sit in `references/` next to this file, or at `https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-migrate/references/<file>`.

| File | When |
|------|------|
| `sources.md` | Gathering material: which docs URLs return content, sibling products, what to read closely |
| `v6-to-v7.md` | Range crosses 7.0.0: kebab-case CSS, new themes, FontAwesome, Button colours, moved classes, `*Data` props, the CSS codemod |
| `v7-to-v8.md` | Range crosses 8.0.0: Scheduler Pro → Scheduler rename, Angular/Vue 2/UMD blockers, headline-change checklist |
| `plan-template.md` | Writing the plan file |
| `worked-example.md` | A real Gantt 6.0.3 → 7.2.1 hop, to calibrate scope |

## Gotchas

- **All `@bryntum/*` product and wrapper packages must be on the same exact version** (`@bryntum/*-lib` helpers aside). A mismatch throws `CSS version X doesn't match bundle version Y` at runtime. A mismatch that already exists is the first plan item. Helpers such as `@bryntum/gantt-lib` (1.0.1 under Gantt 7.3.7) have their own version numbers and come in as dependencies of the main package. `@bryntum/demo-resources` (in apps started from a Bryntum demo) is the reverse: it had its own `1.x` numbers until 6.x and follows the product version from 7.0.0, so bump it to the target too.
- **Take the installed version from `node_modules/@bryntum/<pkg>/package.json`** (or the lockfile), not the semver range in `package.json`. For zip installs, read `build/package.json`.
- **Breaking changes arrive through the products the app inherits from.** A Gantt app breaks on Grid and Scheduler changes too, and Core/Chart changes appear only in their own changelogs. Read the guides for every sibling product, not just the top-level one.
- **Guides are sparse and irregular.** Pre-7.0 guides are rollups (`6.0.0+`), 7.0+ has one per release, and many releases have none. Read every upgrade guide in full. For bug-fix-only releases, list them in the plan and move on.
- **Scheduler Pro stops being a package at 8.0.** It becomes `@bryntum/scheduler` with `tier : 'enterprise'`. The old classes are deprecated aliases until 9.0.0, so the app still runs but logs warnings.
- **A green build doesn't mean the migration is done.** Bryntum deprecations only show at runtime, in the browser console, and some never log at all: Later.js schedule parsing, deprecated in 7.3.0, still runs silently in 7.3.7. List the deprecations named in the guides yourself.
- **Custom CSS can stop matching without being renamed.** The rename tables only cover class renames, not classes that moved to another element. Check custom selectors against the running app (see `v6-to-v7.md`).

## Workflow

### 1. Detect

- **Product and framework** from `@bryntum/*` dependencies. Strip `-trial`, `-thin` and the wrapper suffix (`-react`, `-angular`, `-vue-3`). `-vue` without `-3` is the Vue 2 wrapper, and `-angular-view` is the Angular ≤ 11 View Engine wrapper. If several products are installed (for example Gantt + Grid), run the workflow for the most specific one, because its sibling list covers the rest.
- **Zip install**: there's no `@bryntum/*` dependency, but a local `build/package.json` exists. The zip ships `docs/`, `changelog.md` and a `migrate.js` codemod.
- **Target version**: use the one the user gave. Otherwise use the `latest` tag from `npm view @bryntum/<product> dist-tags`. The `versions` list ends with `-nightly` builds, including 8.0 nightlies, so don't take its last entry. If the licensed registry rejects that call, query `@bryntum/<product>-trial`, which has the same version list. This skill doesn't downgrade. If the target equals the installed version, there's nothing to do. For a prerelease target, warn that it has no guides yet and confirm with the user before continuing.
- **Framework wrapper compatibility**: check the target wrapper's `package.json` and README. `peerDependencies` is empty for `@bryntum/gantt-angular` 7.3.7, so the README line ("Angular: `12` with Ivy support or higher") is the real constraint. `-angular-view` (View Engine) is still published through 7.3.x, but View Engine libraries need `ngcc`, which Angular 16 removed. Apps on Angular 16+ must move to `-angular`.
- For a target ≥ 8.0.0, check the package rename and the environment blockers in `v7-to-v8.md`.
- Tell the user what you found in one block: product (plus any rename), framework, installed → target, sibling products, package manager. Then continue.

### 2. Gather

Follow `sources.md`. The migration index isn't published yet, so expect to use the prerendered docs URLs listed there. Add the hop-specific notes from `v6-to-v7.md` / `v7-to-v8.md`. Turn what you read into candidate items: `{ version, product, source URL + heading, kind, old, new, grep terms }`. The kind is one of breaking, behaviour change, deprecation or cosmetic. Grep terms are identifiers that would appear in customer code: config, method, event and class names, CSS selectors, import paths.

### 3. Plan, then stop for approval

Grep the customer's source (excluding `node_modules` and build output) for each item's terms and mark it *applies* (list the files), *not used*, or *unsure*. Use *unsure* for dynamic access, string-built keys, spread configs and generic terms. Write the plan using `plan-template.md`.

- **Grep more than the source.** Data files (`.json`: `*Data` keys, `iconCls`, Later.js schedule strings), HTML templates (`.html`, `.ejs`) and build config (`angular.json` `styles`, `package.json` scripts such as a `postinstall` that copies Bryntum CSS) often hold the hits. Skip gitignored copies of Bryntum files (theme CSS copied into `public/`) and your own scratch folder, or they drown the results.
- **New defaults have no grep terms.** Guides also announce defaults that changed ("DependencyMenu is now enabled by default", "Dragging all selected tasks by default"). They affect every app that uses the feature, so list each one under *Default behaviour changes* in the plan with its opt-out, instead of marking it *not used*.

Two cheap checks before asking for approval, neither of which touches the project:

- **Baseline the current app.** Run it and capture screenshots, plus the computed styles of custom-styled elements. You need this to compare against after the upgrade, and it also shows which custom rules already didn't match. Write the script's selectors to survive the v7 kebab-case renames (`v6-to-v7.md`).
- **Inspect the target package.** `npm pack @bryntum/<product>@<target>` into a scratch folder, then check CSS file names, the `fontawesome/` path and whether the configs and classes the plan relies on exist, by grepping its `.d.ts` / `.module.js`. For one exact version this is faster and more authoritative than the docs MCP. Also check every path the app reaches into a `@bryntum/*` package by name: SCSS `@import`s, deep imports, `postinstall` copy maps. No guide covers these; `@bryntum/demo-resources/scss/example-vite.scss`, for example, is gone in 7.x (`npm pack --dry-run` lists the files without downloading).

Don't edit anything until the user approves the plan, and that includes `package.json`. Ask with `AskUserQuestion` if it's available, or ask them to review the file and reply "approve". The user may edit the plan before approving, so re-read the file afterwards and apply what it says, not your memory of it. Write the user's answers into the plan under each question. When an answer changes an action (for example, bundler imports instead of copied CSS files), rewrite that action's New code before applying it. Leave unanswered *unsure* items unapplied.

### 4. Apply

- **Zip install:** the user downloads the target zip and replaces `build/` (and `docs/`), then re-runs this skill so the versions are detected again.
- For a Scheduler Pro app going to 8.x, do the package rename first (`v7-to-v8.md`).
- Set every `@bryntum/*` dependency except `*-lib` helpers to the exact target version, with no `^` or `~`, and keep any `@npm:@bryntum/<product>-trial` alias. A `*-lib` helper follows the main package, so if the app lists it directly, drop the pin or use the version the target package depends on. Update `postinstall` scripts that copy Bryntum files first, since the install runs them. Install with the project's package manager, then confirm every non-`*-lib` package in `node_modules` reports the target version.
- For a 6 → 7 hop, run the CSS codemod if it's available (the index's `tools.migrate6to7` or a zip's `migrate.js`), dry-run first. Otherwise handle the selector renames as plan actions (`v6-to-v7.md`).
- Apply the plan's actions in order and tick each one off in the plan file as you go. Skip items marked *not applicable*.
- Move CSS and theme imports to the target layout described in the `bryntum` skill's CSS setup section.
- Restart any running dev server after installing, so the bundler re-optimises `@bryntum/*`. A server that stays running keeps serving the old pre-bundled deps.

### 5. Verify

- Run the project's build/typecheck and tests. Fix errors the migration caused, and report unrelated ones separately. If there are no tests, script the app's own UI actions (toolbar buttons, editors, menus) in a headless browser.
- Compare against the baseline screenshots. v7 themes can change font size and padding, so check fixed-width columns for truncation, and check custom renderer output.
- Grep for every identifier the plan removed or renamed. There should be no hits.
- If a dev server can run, check the browser console for `Deprecation warning`. Bryntum's `VersionHelper.deprecate('<removal version>', msg)` logs `Deprecation warning: You are using a deprecated API which will change in v<version>. <message>`. Once that version ships, the same call throws `Deprecated API use. <message>`. Add each warning to the plan as a follow-up. From 7.3.0, missing structural or theme CSS is loaded from a CDN with a console warning, so a correctly styled page doesn't prove the CSS imports are right; that warning must be absent too. Warnings only fire on code paths that run, so a page-load check alone is partial.
- Stop every dev server you started. One started through `npx` can keep listening after its shell is stopped, so check the port and kill the process if needed.
- Report the applied items, the remaining manual and *unsure* items, and the build/test status. Remind the user to check the console after the first real run.
