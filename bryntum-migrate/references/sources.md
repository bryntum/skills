# Where the upgrade material lives

**Docs host:** `$BRYNTUM_DOCS_BASE_URL` if set (no trailing slash, e.g. `http://127.0.0.1:4179`), otherwise `https://bryntum.com`. Every URL below is `<docs host>/products/<product>/...`. When the host is overridden and the index's `baseUrl` / `docs*BaseUrl` values still start with `https://bryntum.com`, swap that origin for `<docs host>` before fetching anything relative to them.

Put the host and the source you used in the plan header (`Source: prerendered docs (index 404)` or `Source: index.json`).

## Current path: prerendered docs

The rendered guide pages (`<docs host>/products/<product>/docs/guide/<Product>/...`) are client-rendered. Every one returns the same HTML shell with no guide text, so they're no use with `curl` or WebFetch. Use these instead:

| Need | Source |
|------|--------|
| Which guides exist | `<docs host>/products/<product>/docs/data/navigation.js`. Grep it for `guides/upgrades/` and `guides/whats-new/`. Ids look like `<Product>/guides/upgrades/7.0.0.md`, and a product's file also lists the guides of the products it inherits from. |
| Upgrade guide | `<docs host>/products/<product>/docs/ssr/guide/<Product>/upgrades/<version>.html` (rollups too: `.../upgrades/6.0.0+.html`) |
| What's new | `<docs host>/products/<product>/docs/ssr/guide/<Product>/whats-new/<version>.html` |
| Version history (all changelogs, one large page) | `<docs host>/products/<product>/docs/ssr/guide/<Product>/changelog.html` |
| Release versions | `npm view @bryntum/<product> versions --json`, keep `(installed, target]` |
| API diff table | `<docs host>/products/<product>/docs/?v=<version>#apidiff` |
| v7 CSS migration guide (markdown) | `<docs host>/products/<product>/docs-llm/guide/<Product>/migration/migrate-to-new-css.md` |
| Zip install | `docs/` + `changelog.md` inside the extracted archive |

Each page is a full HTML document, and the guide text ends where `<div class="right-pane">` begins. Cut at that string, not at the line: some pages (e.g. Gantt `upgrades/7.1.0.html`) are a single line, so a line-based `sed` deletes the whole guide. `perl -0777 -pe 's/<div class="right-pane">.*//s' page.html | pandoc -f html -t gfm --wrap=none` works if pandoc is installed; otherwise strip the tags. If the output is only a line or two, re-check the raw HTML. Split `changelog.html` on its `## <x.y.z>` release headings and `### <SECTION>` headings. The raw HTML has literal `[BREAKING]` / `[DEPRECATED]` markers, but pandoc escapes them to `\[BREAKING\]`, so match `\\?\[BREAKING\\?\]` after conversion.

Note that the `navigation.js` ids say `guides/` while the prerendered path says `ssr/guide/`. Guessed markdown equivalents (`docs-llm/guide/<Product>/upgrades/...md`, `.../changelog.md`) 404.

Guides exist sparsely — a 404 for a version is normal. Pre-7.0 guides are rollups named `6.0.0+`, `5.0.0+`, … with `## <Product> v6.2.0`-style headings; fetch `.../upgrades/6.0.0+.html` and slice by heading (Scheduler Pro headings read `Scheduler Pro v6.2.0`, with a space), keeping only versions in `(installed, target]`. From 7.0.0 there is one file per release (8.x uses the same URL shapes; a former Scheduler Pro app reads them under `/products/scheduler/`). Repeat for every sibling product (Grid guides live at `/products/grid/docs/ssr/guide/Grid/...`).

Without the index you need the sibling list yourself (`navigation.js` also shows it). Use the installed product for the 7.x part of the range and the target product for the 8.x part:

| App product | Releases < 8.0.0 | Releases ≥ 8.0.0 |
|-------------|------------------|------------------|
| Gantt | Grid → Scheduler → SchedulerPro → Gantt | Grid → Scheduler → Gantt |
| SchedulerPro (→ Scheduler in 8) | Grid → Scheduler → SchedulerPro | Grid → Scheduler |
| Scheduler | Grid → Scheduler | Grid → Scheduler |
| Calendar | Grid → Scheduler → Calendar | Grid → Scheduler → Calendar |
| Grid | Grid | Grid |
| TaskBoard | Grid → Scheduler → TaskBoard | Grid → Scheduler → TaskBoard |

No SchedulerPro guides exist for 8.x. Core and Chart have no upgrade guides.

## What to read, and how closely

Reading list: every release in `(installed, target]`, ascending; per release, every sibling product base-first; per product, whatever guides and changelog sections exist.

| Priority | Read | Why |
|----------|------|-----|
| 1 | Every upgrade guide, fully | Breaking changes and the Old/New code live here |
| 1 | Every changelog with `[BREAKING]` entries (flagged `"breaking": true` in the index) — the sections other than `api-changes` / `styling-changes`, when you already have those from the digests | Far tighter than "has `api-changes`" — mark these releases ✱ in the plan |
| 2 | `api-changes` + `styling-changes` (digests, or those sections); `[DEPRECATED]` entries | Renames, removals, deprecations, CSS class/theme changes not in a guide |
| 3 | What's-new, headings first; read a section only when it names something the app uses or works around | New features that replace a customer workaround or override |
| 4 | `bug-fixes` | Only when customer code references an issue number or carries a workaround comment |
| — | Releases with only `bug-fixes` / `demos` / `locale-updates` and no guide | List in the plan; don't read |

Changelogs have no "BREAKING CHANGES" section — breaking changes are in `API CHANGES`, `STYLING CHANGES`, and the upgrade guides. Entries look like `* Fixed #12206 - ...`.

Sibling products repeat each other: the same section (e.g. 7.3.0 "`content: var(--fa)` removed") appears word for word in the Grid, Scheduler, Scheduler Pro and Gantt guides. Skip a section only after confirming its full content repeats one already read, not just its heading. For example, Grid and Scheduler 7.0.0 both have "Individual animation related configs deprecated", but Scheduler adds `enableEventAnimations` → `transition.changeEvent`, which Grid's section doesn't cover. The reading-list table in the plan gets long (100+ rows for Gantt 6.0 → 7.3); collapse consecutive releases that only have bug fixes into one row per product.

## Future path: the migration index

`curl -sf <docs host>/products/<product>/docs-llm/migration/index.json`, e.g. `https://bryntum.com/products/gantt/docs-llm/migration/index.json`. It isn't published on bryntum.com yet (404 for Gantt and Grid as of 2026-09-29), but it may be on a `$BRYNTUM_DOCS_BASE_URL` host. It costs one request to try. On a 200, prefer it to the prerendered pages. Schema:

```json
{
  "schemaVersion": 1,
  "product": { "id": "gantt", "name": "Gantt" },
  "baseUrl": "https://bryntum.com/products/gantt/docs-llm/migration/",
  "docsApiBaseUrl": "https://bryntum.com/products/gantt/docs/api/",
  "docsGuideBaseUrl": "https://bryntum.com/products/gantt/docs/guide/",
  "docsLlmGuideBaseUrl": "https://bryntum.com/products/gantt/docs-llm/guide/",
  "products": ["Gantt", "SchedulerPro", "Scheduler", "Grid", "Chart", "Core"],
  "productOrder": "most-specific first; read guides for a version in reverse order (base product first)",
  "guideCoverage": {
    "Gantt": { "upgrades": true, "whatsNew": true }, "SchedulerPro": { "upgrades": true, "whatsNew": true },
    "Scheduler": { "upgrades": true, "whatsNew": true }, "Grid": { "upgrades": true, "whatsNew": true },
    "Chart": { "upgrades": false, "whatsNew": false }, "Core": { "upgrades": false, "whatsNew": false }
  },
  "digests": { "Gantt": "Gantt/changelog/api-changes.md", "Grid": "Grid/changelog/api-changes.md", "Core": "Core/changelog/api-changes.md" },
  "tools": { "migrate6to7": "tools/migrate.js", "selectors": "tools/selectors.md" },
  "releases": [
    { "version": "7.2.1", "date": "2026-02-26", "products": {
        "Gantt": { "changelog": { "url": "Gantt/changelog/7.2.1.md", "sections": ["demos", "bug-fixes"] } },
        "Grid":  { "changelog": { "url": "Grid/changelog/7.2.1.md",  "sections": ["api-changes", "bug-fixes"] } }
    } },
    { "version": "6.2.0", "date": "2025-04-10", "products": {
        "Gantt": {
          "changelog": { "url": "Gantt/changelog/6.2.0.md", "sections": ["features-enhancements", "api-changes", "bug-fixes"], "breaking": true },
          "upgrade":   { "url": "Gantt/upgrades/6.2.0.md",  "source": "Gantt/upgrades/6.0.0+.md",  "anchor": "Gantt v6.2.0" },
          "whatsNew":  { "url": "Gantt/whats-new/6.2.0.md", "source": "Gantt/whats-new/6.0.0+.md", "anchor": "Gantt v6.2.0" }
        } } }
  ]
}
```

Things that aren't obvious from the schema:

- All `url` values are relative to `baseUrl`. `releases` is sorted descending.
- `products` is most-specific first and includes the base libraries (Core, Chart). For each version, read guides base product first. Core and Chart contribute changelogs and `digests` only, never guides — but their digests carry Store / Model / DomHelper / CSS-variable breaking changes that are never mirrored into product changelogs, so don't skip them.
- Start with the `digests`: each holds only API CHANGES + STYLING CHANGES, for every release back to 1.x (Grid's is ~64 KB). Cut it to `(installed, target]` on its `## <x.y.z> - <date>` headings. It leaves out `[BREAKING]` / `[DEPRECATED]` entries filed under FEATURES / ENHANCEMENTS (e.g. the 7.3.6 UMD bundle deprecation), so still read those sections of the flagged changelogs.
- `guideCoverage.<Product>.upgrades === false` means changelog-only in this index (Core, Chart, and SchedulerPro on the 8.x line). A missing `upgrade` key for such a product doesn't mean "no changes this version" — SchedulerPro 8.x changes live in the Scheduler 8.x guides.
- `upgrade` / `whatsNew` are already sliced per version from the rollups. A slice can still contain several `## <Product> v<x.y.z>` sections (a version appearing twice in a rollup, or Scheduler + Scheduler Pro sharing a file on 8.x) — read the whole slice. Multi-section slices carry `anchors` (every heading, in file order) and, when several rollups contributed, `sources`. On an 8.x-line index, an `anchors` entry reading `Scheduler Pro …` is where the former Scheduler Pro changes live.
- Per-release 7.x/8.x guides have no `source`/`anchor`; `date` is `null` for guide-only versions.
- `sections` ids: `features-enhancements`, `api-changes`, `styling-changes`, `locale-updates`, `demos`, `bug-fixes`. `"breaking": true` / `"deprecated": true` (omitted when false) come from `[BREAKING]` / `[DEPRECATED]` entry markers — the tightest filter available.
- `tools.migrate6to7` and `tools.selectors` are present only when the files exist.
- Relative links inside a guide: `(#Gantt/model/ProjectModel#config-x)` → `docsApiBaseUrl + "Gantt/model/ProjectModel#config-x"`. A guide link `(#<Product>/guides/<path>.md)` lives under the **linked** product's host, with `guides/` dropped: `(#Grid/guides/migration/migrate-to-new-css.md)` → `<docs host>/products/grid/docs-llm/guide/Grid/migration/migrate-to-new-css.md`. Keeping `guides/`, or using the app's own product host for a sibling's guide, returns 404.
