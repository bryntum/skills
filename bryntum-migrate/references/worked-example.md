# Worked example — Gantt 6.0.3 → 7.2.1, React + Vite

Useful for calibrating how sparse the guides are and how much a real hop involves.

Detection: `@bryntum/gantt` and `@bryntum/gantt-react` resolve to `6.0.3` in `node_modules`; `package-lock.json` → npm; target `7.2.1` (< 8, so no package rename); siblings Grid → Scheduler → SchedulerPro → Gantt. 33 releases in `(6.0.3, 7.2.1]`. Guides in range, per product (rollup `6.0.0+` sections count as one guide each):

| Product | Upgrade guides | What's-new guides |
|---------|----------------|-------------------|
| Grid | 6.1.2, 6.1.6, 6.2.0, 6.2.4, 7.0.0, 7.0.1, 7.0.2, 7.1.0, 7.2.0 | 6.1.0, 6.1.2, 6.1.4, 6.1.6, 6.1.7, 6.1.8, 6.2.0, 6.3.0, 7.0.0, 7.1.0, 7.1.1, 7.2.0 |
| Scheduler | 6.1.6, 6.1.8, 6.2.0, 6.3.0, 7.0.0, 7.1.0 | 6.0.5, 6.1.0, 6.1.1, 6.1.4, 6.1.8, 6.2.0, 6.3.0, 7.0.0, 7.2.0 |
| SchedulerPro | 6.2.0, 6.3.0, 7.0.0, 7.1.0, 7.2.0 | 6.0.5, 6.1.0, 6.1.3, 6.1.4, 6.1.7, 6.1.8, 6.2.0, 6.2.4, 6.3.0, 7.0.0, 7.2.0 |
| Gantt | 6.1.6, 6.2.0, 6.3.0, 7.0.0, 7.1.0, 7.2.0 | 6.1.0, 6.1.3, 6.1.4, 6.1.6, 6.1.7, 6.1.8, 6.2.0, 6.2.3, 6.2.4, 6.2.5, 6.3.0, 6.3.1, 7.0.0, 7.2.0 |

There is no Gantt 6.1.0 upgrade guide — 6.1.0 has a what's-new entry and a changelog only. Reading list, Gantt rows (excerpt; sibling rows follow the same shape, `versions-support` omitted — it is never read; ✱ = changelog carries `[BREAKING]` entries):

| Version | Product | Upgrade guide | What's new | Changelog sections (✱ = BREAKING) |
|---------|---------|---------------|------------|-----------------------------------|
| 6.0.4 – 6.0.5 | Gantt | — | — | features-enhancements / demos, bug-fixes (listed, not read) |
| 6.0.6 | Gantt | — | — | **api-changes**, demos, bug-fixes |
| 6.1.0 | Gantt | — | `## Gantt v6.1.0` in `whats-new/6.0.0+` | features-enhancements, demos, bug-fixes |
| 6.1.6 | Gantt | `## Gantt v6.1.6` in `upgrades/6.0.0+` | `## Gantt v6.1.6` | features-enhancements, bug-fixes |
| 6.2.0 | Gantt | `## Gantt v6.2.0` | `## Gantt v6.2.0` | ✱ features-enhancements, **api-changes**, demos, bug-fixes |
| 6.3.0 | Gantt | `## Gantt v6.3.0` | `## Gantt v6.3.0` | features-enhancements, **api-changes**, locale-updates, demos, bug-fixes |
| 7.0.0 | Gantt | `upgrades/7.0.0` | `whats-new/7.0.0` | ✱ features-enhancements, **api-changes**, **styling-changes**, locale-updates, demos, bug-fixes |
| 7.0.1 – 7.0.2 | Gantt | — (Grid has 7.0.1 and 7.0.2 guides) | — | bug-fixes (listed, not read) |
| 7.1.0 | Gantt | `upgrades/7.1.0` ("Salesforce support") | — | features-enhancements, **styling-changes**, bug-fixes |
| 7.1.1 – 7.1.3 | Gantt | — | — | bug-fixes / demos (listed, not read) |
| 7.2.0 | Gantt | `upgrades/7.2.0` ("Prevent silencing resolved scheduling issues") | `whats-new/7.2.0` | features-enhancements, demos |
| 7.2.1 | Gantt | — | — | demos, bug-fixes (listed, not read) |

One action item from the plan, taken verbatim from the Gantt 7.0.0 upgrade guide and adapted to the customer's file:

````markdown
### 7.0.0 — Gantt
- [ ] **Rename project config `autoPostponedConflicts` → `autoPostponeConflicts`** · risk: deprecation (old name still works, deprecated)
  Source: https://bryntum.com/products/gantt/docs/guide/Gantt/upgrades/7.0.0 → "Project `autoPostponedConflicts` has been renamed"
  Files: `src/components/ProjectGantt.tsx`
  **Old code**
  ```tsx
  <BryntumGantt project={{ autoPostponedConflicts: true }} />
  ```
  **New code**
  ```tsx
  <BryntumGantt project={{ autoPostponeConflicts: true }} />
  ```
````

Other 7.0.0 guide headings the plan must answer for this app: Gantt — "Assignment field picker has changed to TabPanel", "Dragging all selected tasks by default", "Tracking project settings changes by default", "Resizable task editor", "Conflicts postponing related changes" (`hasPostponedOwnConstraintConflict` → `postponedConflict`); Grid — "New themes & styling changes", "FontAwesome Free no longer built in", "Individual animation related configs deprecated", "Container layout changes", "Mask mode config removed"; Scheduler — "The DependencyMenu feature is now enabled by default", "Dragged events now snap to resources by default", "Event styles changed". From 6.3.0: "Naming simplification for project data properties" (`tasksData` → `tasks`, …). The theme replacement lands under Unknowns for the user (old `gantt.stockholm.css` is gone; propose `svalbard-light.css`, the v7 default; `stockholm-light.css` keeps today's look; Visby, Material3, High Contrast, Fluent2 also ship in `-light`/`-dark`), and the kebab-case selector rename is handled by the codemod dry-run before any manual CSS edits.
