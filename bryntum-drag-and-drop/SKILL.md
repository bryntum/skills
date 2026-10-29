---
name: bryntum-drag-and-drop
description: >
  Drag-and-drop INTO a Bryntum Scheduler (any tier, incl. the former Scheduler Pro) / Gantt from an outside source —
  a sidebar of unscheduled jobs, plain <div>s, a third-party grid (AG Grid), or a Bryntum
  Grid. Use alongside the `bryntum` skill whenever items are dragged onto the timeline.
  Trigger on phrases like "drag from sidebar", "drag unscheduled tasks", "drag onto the
  scheduler", "external drag source", "DragHelper", or a drop being "rejected"/"not working".
metadata:
  tags: bryntum, drag, drop, draghelper, scheduler, schedulerpro, gantt, external-source, grid
---

## Version check — do this first

Detect the version as the `bryntum` skill's Version check says (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md). Every code block in this file is 8.0 code. 7.x → also load `references/v7.md` (raw: https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum-drag-and-drop/references/v7.md).

---

## dragging from another Bryntum Grid/store

On 8.x, if the drag source IS a Bryntum Grid/store, use the official store→store demo — that's what its `Drag.js` is built for: https://bryntum.com/products/scheduler/examples/drag-from-grid/app.module.js, https://bryntum.com/products/scheduler/examples/drag-from-grid/lib/Drag.js The demo runs at the Enterprise tier (`taskEdit`, `stickyEvents`); on Community/Business drop those feature blocks — the `Drag.js` class itself needs nothing above Community.

## Dragging from an external (non-Bryntum) source

Core use case: a sidebar/pool of unscheduled jobs that the user drags onto a resource row in the Scheduler.

Use Bryntum's [`DragHelper`](https://bryntum.com/products/scheduler/docs/api/Core/helper/DragHelper) and point it at your OWN DOM elements, reading data from your OWN source. `DragHelper` works with any draggable HTML element — it is not limited to Bryntum stores.

### Minimal pattern

```ts
import { DragHelper } from '@bryntum/core-enterprise';
import { Scheduler } from '@bryntum/scheduler-enterprise';

class JobDragHelper extends DragHelper {
    static configurable = {
        callOnFunctions    : true,                 // call onDragStart/onDrag/onDrop methods
        cloneTarget        : true,                 // drag a visual copy, not the original
        dropTargetSelector : '.b-timeline-sub-grid', // WHERE you may drop (the scheduler timeline). v7 kebab-case; pre-rename versions used '.b-timeline-subgrid'
        targetSelector     : '.unscheduled-job'    // WHAT can be dragged (YOUR sidebar element)
    };

    constructor(config) {
        super({ outerElement : config.outerElement }); // the container holding your draggable elements
        this.scheduler = config.scheduler;
    }

    // Build the drag proxy from YOUR data (look the record up by an id on the element)
    createProxy(grabbedElement) {
        const job   = this.getJobFor(grabbedElement);  // your lookup: element id -> your JS record
        const proxy = document.createElement('div');
        proxy.classList.add('b-sch-event-wrap', 'b-style-bordered', 'b-sch-horizontal');
        proxy.innerHTML = `<div class="b-sch-event"><div class="b-sch-event-content">${job.name}</div></div>`;
        this.context.job = job;
        return proxy;
    }

    onDragStart({ context }) {
        context.job = this.getJobFor(context.grabbed);
    }

    // Validate continuously: is there a valid date + resource under the cursor?
    onDrag({ context }) {
        const date     = this.scheduler.getDateFromCoordinate(context.newX, 'round', false);
        const resource = context.target && this.scheduler.resolveResourceRecord(context.target);
        context.valid    = Boolean(date && resource);
        context.date     = date;
        context.resource = resource;
    }

    // Commit to the Bryntum store on a valid drop
    async onDrop({ context }) {
        if (!context.valid) return;
        const { job, date, resource } = context;
        await this.scheduler.scheduleEvent({
            eventRecord    : this.scheduler.eventStore.getById(job.id) ?? { id: job.id, name: job.name, duration: job.duration },
            startDate      : date,
            resourceRecord : resource
        });
        // remove the job from YOUR source list (state/store/DOM)
    }
}
```

On 8.x `DragHelper` comes from `@bryntum/core-<tier>` and `Scheduler` from `@bryntum/scheduler-<tier>`; the former Scheduler Pro is Scheduler at the Enterprise tier.

Instantiate after the component exists (in React, inside an effect once you have the instance):

```ts
const drag = new JobDragHelper({
    scheduler    : schedulerInstance,
    outerElement : document.querySelector('.sidebar') // container of your .unscheduled-job elements
});
// clean up on unmount: drag.destroy();
```

### Hook reference

- `targetSelector` — your own draggable elements (any DOM, not a Bryntum store).
- `dropTargetSelector` — restrict drops to the timeline: `'.b-timeline-sub-grid'` (v7 kebab-case CSS; pre-rename versions used `'.b-timeline-subgrid'`).
- `createProxy(el)` — build the visual drag proxy from your data; stash the record on `this.context`.
- `onDragStart` — capture the dragged record into `context`.
- `onDrag` — per-move validation; set `context.valid`. Use `scheduler.getDateFromCoordinate()` and `scheduler.resolveResourceRecord(context.target)`.
- `onDrop` — if `context.valid`, commit via `scheduler.scheduleEvent({ eventRecord, startDate, resourceRecord })` (or `scheduler.eventStore.assignEventToResource(event, resource)` for an event that already has dates), then remove the item from your source.

For richer, in-scheduler drag validation also add the `eventDrag` feature's `validatorFn`.

### Full worked example

**Drag from an AG Grid into Bryntum Scheduler Pro** — a complete React + Express implementation of this exact pattern (custom `DragHelper`, proxy sized to duration, role/calendar validation, removal from the source grid, backend sync):
- Blog: https://bryntum.com/blog/how-to-integrate-a-react-ag-grid-with-a-react-bryntum-scheduler-pro/

The same article also shows keeping the source list in sync with the Bryntum stores via store events (`add`/`remove`/`update`).

> **Version caveat:** that article targets Bryntum **v6**, so its CSS class names are pre-rename (e.g. `.b-timeline-subgrid`). v7 normalized classes to kebab-case (`.b-timeline-sub-grid`, `.b-grid-sub-grid`, `.b-time-axis-sub-grid`, `.b-button-group`, …). Copy the *pattern/logic* from the article but translate any CSS class names to their v7 kebab-case form, and on 8.x use the tiered packages and owning-package imports above. See https://bryntum.com/products/scheduler/docs-llm/guide/Scheduler/migration/migrate-to-new-css.md
