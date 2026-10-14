# Where the upgrade material lives

**Docs host:** `$BRYNTUM_DOCS_BASE_URL` if set (no trailing slash, e.g. `http://127.0.0.1:4179`), otherwise `https://bryntum.com`. Every URL below is `<docs host>/products/<product>/...`. When the host is overridden and URLs from the docs (the index's `baseUrl` / `docs*BaseUrl`, `llms.txt` links) still start with `https://bryntum.com`, swap that origin for `<docs host>`. The host's `/examples/` and `/build/` serve the current release, not your target: don't use them as a reference.

Put the host and the source you used in the plan header (`Source: index.json` or `Source: prerendered docs (index 404)`).

Use the **target** product's host: a former Scheduler Pro app going to 8.x reads `/products/scheduler/` (there is no Scheduler Pro index for 8.x, and the Scheduler guides hold the merged Scheduler Pro sections).

## First choice: the migration index

`curl -sf <docs host>/products/<product>/docs-llm/migration/index.json`. One request tells you whether it exists; on a 404 (or an HTML page instead of JSON), use the prerendered docs below. Schema:

```json
{
  "schemaVersion": 1,
  "product": { "id": "gantt", "name": "Gantt" },
  "baseUrl": "https://bryntum.com/products/gantt/docs-llm/migration/",
  "docsApiBaseUrl": "https://bryntum.com/products/gantt/docs/api/",
  "docsLlmGuideBaseUrl": "https://bryntum.com/products/gantt/docs-llm/guide/",
  "products": ["Gantt", "Scheduler", "Grid", "Chart", "Core"],
  "guideCoverage": { "Gantt": { "upgrades": true, "whatsNew": true }, "Core": { "upgrades": true, "whatsNew": false }, "Chart": { "upgrades": false, "whatsNew": false } },
  "digests": { "Gantt": "Gantt/changelog/api-changes.md", "Core": "Core/changelog/api-changes.md" },
  "tools": { "migrate6to7": "tools/migrate.js", "selectors": "tools/selectors.md" },
  "releases": [
    { "version": "8.0.0", "date": "2026-10-12", "products": {
        "Gantt": {
          "changelog": { "url": "Gantt/changelog/8.0.0.md", "sections": ["features-enhancements", "api-changes", "bug-fixes"], "breaking": true },
          "upgrade":   { "url": "Gantt/upgrades/8.0.0.md" },
          "whatsNew":  { "url": "Gantt/whats-new/8.0.0.md" }
        },
        "Core": { "upgrade": { "url": "Core/upgrades/8.0.0.md" } } } },
    { "version": "6.2.0", "date": "2025-04-10", "products": {
        "Gantt": { "upgrade": { "url": "Gantt/upgrades/6.2.0.md", "source": "Gantt/upgrades/6.0.0+.md", "anchor": "Gantt v6.2.0" } } } }
  ]
}
```

Things that aren't obvious from the schema:

- All `url` values are relative to `baseUrl`. `releases` is sorted descending; keep `(installed, target]`.
- `products` is most-specific first and includes Core and Chart. For each version, read base product first. Core has an upgrade guide from 8.0.0 on; Chart has none. Core and Chart changelogs carry Store / Model / DomHelper / CSS-variable changes that are never mirrored into product changelogs, so don't skip them.
- **Digests** (`digests`) hold every release's API CHANGES and STYLING CHANGES plus the `[BREAKING]` / `[DEPRECATED]` entries of the other sections, back to 1.x. Cut them to `(installed, target]` on their `## <x.y.z> - <date>` headings. They pay off for multi-release ranges; for a one-release hop read that release's changelogs instead.
- `"breaking": true` / `"deprecated": true` (omitted when false) mark changelogs with `[BREAKING]` / `[DEPRECATED]` entries, the tightest filter available. Some changes are only there, not in a guide (8.0.0 `smoothZoom`, `zoomFactor`).
- `guideCoverage.<Product>.upgrades === false` means changelog-only. A missing `upgrade` key doesn't mean "no changes this version".
- `upgrade` / `whatsNew` are already sliced per version from the pre-7.0 rollups. A slice can hold several `## <Product> v<x.y.z>` sections (`anchors`, `sources`): read the whole slice.
- `sections` ids: `features-enhancements`, `api-changes`, `styling-changes`, `locale-updates`, `demos`, `bug-fixes`.
- `tools.migrate6to7` and `tools.selectors` are present only when the files exist.
- Relative links inside a guide: `(#Gantt/model/ProjectModel#config-x)` → `docsApiBaseUrl + "Gantt/model/ProjectModel#config-x"`. A guide link `(#<Product>/guides/<path>.md)` lives under the **linked** product's host with `guides/` dropped: `<docs host>/products/grid/docs-llm/guide/Grid/migration/migrate-to-new-css.md`.

Other docs-llm pages that answer questions the guides don't: `docs-llm/api/<Product>/<path>/<Class>.md` (API docs, incl. `@deprecated`), `docs-llm/guide/<Product>/data/tiers.md` (8.x tiers), `docs-llm/guide/<Product>/integration/<framework>/guide.md`, and `docs-llm/llms.txt` (the full list).

## Fallback: prerendered docs

The client-rendered guide pages (`/docs/guide/...`) return an empty HTML shell, so use:

| Need | Source |
|------|--------|
| Which guides exist | `/docs/data/navigation.js`, grep for `guides/upgrades/` and `guides/whats-new/` (ids like `<Product>/guides/upgrades/7.0.0.md`). It misses Core's 8.0.0 guide; fetch that one directly. |
| Upgrade guide | `/docs/ssr/guide/<Product>/upgrades/<version>.html` (rollups: `.../upgrades/6.0.0+.html`) |
| What's new | `/docs/ssr/guide/<Product>/whats-new/<version>.html` |
| All changelogs (one large page) | `/docs/ssr/guide/<Product>/changelog.html` |
| v7 CSS migration guide | `/docs-llm/guide/<Product>/migration/migrate-to-new-css.md` |
| Zip install | `docs/` + `changelog.md` inside the extracted archive |

The guide text ends where `<div class="right-pane">` begins. Cut at that string, not at a line (some pages are a single line): `perl -0777 -pe 's/<div class="right-pane">.*//s' page.html | pandoc -f html -t gfm --wrap=none`, or strip the tags. Split `changelog.html` on its `## <x.y.z>` and `### <SECTION>` headings; pandoc escapes `[BREAKING]` to `\[BREAKING\]`. Guides exist sparsely, so a 404 for a version is normal. Pre-7.0 rollups have `## <Product> v6.2.0` headings (Scheduler Pro: `Scheduler Pro v6.2.0`); slice them to `(installed, target]`.

Sibling products, base first (the index's `products` gives the same list, plus Chart, whose changelogs count too):

| App product | Releases < 8.0.0 | Releases ≥ 8.0.0 |
|-------------|------------------|------------------|
| Gantt | Grid → Scheduler → SchedulerPro → Gantt | Core → Grid → Scheduler → Gantt |
| SchedulerPro (→ Scheduler in 8) | Grid → Scheduler → SchedulerPro | Core → Grid → Scheduler |
| Scheduler | Grid → Scheduler | Core → Grid → Scheduler |
| Calendar | Grid → Scheduler → Calendar | Core → Grid → Scheduler → Calendar |
| Grid | Grid | Core → Grid |
| TaskBoard | Grid → Scheduler → TaskBoard | Core → Grid → Scheduler → TaskBoard |

## What to read, and how closely

Reading list: every release in `(installed, target]`, ascending; per release, every sibling product base-first; per product, whatever guides and changelog sections exist.

| Priority | Read | Why |
|----------|------|-----|
| 1 | Every upgrade guide, fully | Breaking changes and the Old/New code live here |
| 1 | Every changelog flagged breaking or deprecated (✱ in the plan): its `[BREAKING]` / `[DEPRECATED]` entries | Some changes are only in the changelog |
| 2 | `api-changes` + `styling-changes` (digests, or those sections) | Renames, removals, CSS class/theme changes not in a guide |
| 3 | What's-new, headings first; a section only when it names something the app uses or works around | New features that replace a customer workaround |
| 4 | `bug-fixes` | Only when customer code references an issue number or carries a workaround comment |
| — | Releases with only `bug-fixes` / `demos` / `locale-updates` and no guide | List in the plan; don't read |

Sibling products repeat each other: the same section often appears word for word in the Grid, Scheduler and Gantt guides. Skip a section only after confirming its full content repeats one already read, not just its heading. Grid and Scheduler 7.0.0 both have "Individual animation related configs deprecated", but Scheduler's adds `enableEventAnimations` → `transition.changeEvent`. Collapse consecutive bug-fix-only releases into one row per product in the plan's reading list.
