# Migration plan template

Write it to `./bryntum-migration-<from>-to-<to>.md` in the project root. Group actions by version, then by product base-first, so they can be applied top to bottom. Copy each guide's Old code / New code blocks and adapt them to the customer's actual file.

````markdown
# Bryntum migration: <Product> <from> → <to>

Product: <Product> (+ siblings: …) · Framework: <framework> · Package manager: <pm>
Package rename: none | `@bryntum/schedulerpro*` → `@bryntum/scheduler*` (Scheduler Pro is the Enterprise tier of Scheduler from 8.0)
Releases crossed: <n> (<list>) · Upgrade guides read: <n>
Docs host: <docs host> · Source: index.json | docs fallback (index 404)

## Reading list
| Version | Product | Upgrade guide | What's new | Changelog sections (✱ = BREAKING) |
|---------|---------|---------------|------------|-----------------------------------|

## Actions (in version order)
### Packages
- [ ] **Bump every `@bryntum/*` package to exactly `<to>`** (incl. `@bryntum/demo-resources`; not `*-lib` helpers)
  Files: `package.json`, lockfile · first update any `postinstall` script that copies Bryntum files

### <version> — <Product>
- [ ] **<short title>** · risk: breaking | behaviour change | deprecation | cosmetic
  Source: <url> → "<heading>"
  Files: `src/…`, `src/…`
  **Old code**
  ```js
  …
  ```
  **New code**
  ```js
  …
  ```

## Default behaviour changes — keep the new default or opt out?
- <version> · <Product> · "<guide heading>" · opt out: `<config>`

## Unsure — needs your answer
- …
  **Answer:** <filled in after the user replies>

<details><summary>## Not applicable (n items, zero hits in this codebase)</summary>
- <version> · <Product> · <title> · <source>
</details>

## Verification
- [ ] Build / typecheck: `<command>`
- [ ] Grep for removed identifiers returns zero hits: …
- [ ] Tests: `<command>` (if present)
- [ ] Manual: open the app, search the browser console for `Deprecation warning`

## Unknowns for the user
- Theme: v7 replaces `<product>.<old>.css`. I propose `svalbard-light.css`, the v7 default. To keep today's look instead, use the same-named `<old>-light.css`. Other options: visby, material3, high-contrast, fluent2, each `-light` / `-dark`
- Theme look: even `<old>-light` differs from v6 (sentence-case headers and buttons, blue row selection, round icon buttons, wider cell padding, thinner parent bars). Override any of these?
- …
````
