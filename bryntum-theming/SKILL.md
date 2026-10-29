---
name: bryntum-theming
description: >
  Bryntum themes and dark mode. Use alongside the `bryntum` skill whenever the user picks or
  switches a Bryntum theme, wants light/dark mode or a runtime theme toggle, asks the component
  to match a design system (Material / Fluent), or customizes colors via CSS variables. Covers
  all products — Scheduler (incl. the former Scheduler Pro), Gantt, Calendar, Grid, TaskBoard.
  Trigger on phrases like "dark mode", "theme switch", "toggle theme", "color-scheme",
  "DomHelper.setTheme", "DomHelper.colorScheme", "which theme", "match Material UI", or
  "Bryntum theme".
metadata:
  tags: bryntum, theme, theming, dark-mode, color-scheme, domhelper, settheme, css-variables
---

## Version check — do this first

Detect the version as the `bryntum` skill's Version check says (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md). Every code block in this file is 8.0 code. 7.x → also load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-theming/references/v7.md).

---

## Theme catalog

8.x only. On 7.x, `references/v7.md` replaces this section.

Six theme families: `svalbard` (default), `stockholm`, `visby`, `material3`, `fluent2`, `high-contrast`. Each ships as three files in every product package: the **family sheet** (`svalbard.css`), which renders light or dark from the page's `color-scheme`, and two fixed **variants** (`svalbard-light.css`, `svalbard-dark.css`).

Import exactly one theme, from the package of the highest product on the page (`@bryntum/gantt-<tier>/svalbard.css`), after the structural `<product>.css` — see the CSS setup section of the `bryntum` skill. Two themes on one page fight over the same variables.

---

## Choosing a theme

8.x only. On 7.x, `references/v7.md` replaces this section.

To make the Bryntum component blend with a design-system host app, start from a matching family rather than restyling widgets by hand:

- Material UI → `material3`
- Fluent / Microsoft-style → `fluent2`
- Accessibility-first → `high-contrast`
- Otherwise → `svalbard` (default)

Customize further with CSS variables (`--b-widget-background`, etc.), not by overriding individual class rules.

---

## Dark mode

8.x only. On 7.x, `references/v7.md` replaces this section.

Load the family sheet; the mode then follows `color-scheme`, so dark mode needs no theme swap:

- **Follow the system**: `:root { color-scheme: light dark; }`. Scope it to one container instead (`.dashboard { color-scheme: light dark; }`) to leave the rest of the page alone.
- **Pin from code**: `DomHelper.colorScheme = 'dark'` (or `'light'`); `null` returns to what the page declares. `DomHelper` comes from `@bryntum/core-<tier>`. The pin is written inline on `<html>`, so it beats a `color-scheme` set on `:root` from a stylesheet but yields to a scheme declared nearer a widget.
- **Toggle from code**: `DomHelper.toggleLightDarkTheme()` flips the pin on a family sheet, or swaps the `-light` / `-dark` file of a variant (dropping any pin).
- With nothing declared, the family sheet renders light.
- Swapping `-light` / `-dark` variants works but reloads CSS and drops any pin; prefer the family sheet + `color-scheme`.

A fixed variant (`-light` / `-dark`) renders one mode and ignores the pin with a console warning. Use variants only when the mode must never change.

---

## Switching the theme family at runtime

8.x only. On 7.x, `references/v7.md` replaces this section.

`DomHelper.setTheme()` swaps the `<link>` marked `data-bryntum-theme`, so the theme must be loaded via a `<link>` in `index.html`, not a CSS `@import`:

```html
<link rel="stylesheet" href="/node_modules/@bryntum/core-enterprise/fontawesome/css/fontawesome.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/core-enterprise/fontawesome/css/solid.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/gantt-enterprise/gantt.css" />
<link rel="stylesheet" href="/node_modules/@bryntum/gantt-enterprise/svalbard.css" data-bryntum-theme />
```

```javascript
import { DomHelper } from '@bryntum/core-enterprise';

DomHelper.setTheme('stockholm');        // another family sheet — the color-scheme pin stays
DomHelper.setTheme('stockholm-dark');   // a fixed variant — clears any pin
```

**What does NOT work**:
- CSS `@import` + `setTheme()` — no `<link>` to target, silently fails
- Manual CSS overrides — only covers a fraction of theme variables

---

## Related

Customizing what's rendered *inside* event bars (`eventRenderer`, multi-line layouts, event bar padding) is the `bryntum-styling` skill — theming only picks the look of the whole component.
