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
- [ ] **For 8.x, configure the selected target registry in the project and CI before installing** (`v7-to-v8.md`; preserve a supplied local registry)
- [ ] **Rename and bump Bryntum product, wrapper and `@bryntum/demo-resources` packages to exactly `<to>`**
  Files: `package.json`, lockfile · first update any `postinstall` script that copies Bryntum files
  For targets below 8.0, preserve trial aliases and use each target product's required `*-lib` / `*-lib-trial` versions for direct helper pins; for 8.x, remove obsolete direct helper dependencies.
- [ ] **Imports and CSS paths** from the packed target package (owning package per class, FontAwesome, themes)
- [ ] **For 8.x Business or Enterprise, register the license key** (`v7-to-v8.md`; ask for it, never invent one)

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
- [ ] Product, wrapper and `demo-resources` versions match `<to>`; 7.x helper versions match the target product's dependencies
- [ ] Build / typecheck: `<command>`
- [ ] Grep for removed identifiers returns zero hits: …
- [ ] Tests: `<command>` (if present)
- [ ] Manual: open the app, the console has no errors, no `Deprecation warning` and no `CSS issues detected`
- [ ] 8.x Business or Enterprise: no Trial badge and no license message once the key is set (both show until then)

## Unknowns for the user
- Theme choice (`v6-to-v7.md` for 6 → 7), anything the guides leave to the customer (the 8.x licence tier and key go under *Unsure*)
- …
````
