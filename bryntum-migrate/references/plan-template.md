# Migration plan template

Write it to `./bryntum-migration-<from>-to-<to>.md` in the project root. Group actions by version, then by product base-first, so they can be applied top to bottom. Copy each guide's Old code / New code blocks and adapt them to the customer's actual file.

````markdown
# Bryntum migration: <Product> <from> → <to>

Product: <Product> (+ siblings: …) · Framework: <framework> · Package manager: <pm>
Packages: <old> → <new> (from 8.0 every name carries the tier, `v7-to-v8.md`) · Tier: <tier> (the customer's licence)
Releases crossed: <n> (<list>) · Upgrade guides read: <n>
Docs host: <docs host> · Source: index.json | prerendered docs (index 404)
Blockers: none | <Vue 2 wrapper, Angular < 15, UMD, target not published> (stop and ask before planning further)

## Reading list
| Version | Product | Upgrade guide | What's new | Changelog sections (✱ = BREAKING) |
|---------|---------|---------------|------------|-----------------------------------|

## Actions (in version order)
### Packages
- [ ] **Rename and bump every `@bryntum/*` package to exactly `<to>`** (incl. `@bryntum/demo-resources`)
  Files: `package.json`, lockfile · first update any `postinstall` script that copies Bryntum files
- [ ] **Imports and CSS paths** from the packed target package (owning package per class, FontAwesome, themes)

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

## Not applicable (n items, zero hits in this codebase)
<details><summary>Show</summary>

- <version> · <Product> · <title> · <source>
</details>

## Verification
- [ ] Build / typecheck: `<command>`
- [ ] Grep for removed identifiers returns zero hits: …
- [ ] Tests: `<command>` (if present)
- [ ] Manual: open the app, the console has no errors, no `Deprecation warning` and no `CSS issues detected`

## Unknowns for the user
- Licence tier (8.x), theme choice (`v6-to-v7.md` for 6 → 7), anything the guides leave to the customer
- …
````
