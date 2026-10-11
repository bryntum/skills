---
name: bryntum-editor
description: >
  Customize Bryntum's built-in event/task editor popup (all products except Grid). Use alongside the
  `bryntum` skill to add, remove, or reorder editor fields and tabs, tweak validation, react
  to the editor opening/saving, or swap in a fully custom dialog. Trigger on phrases like
  "edit event popup", "task editor", "remove the % Complete field", "custom event editor",
  "eventEdit"/"taskEdit" config, or "beforeEventEdit"/"beforeTaskEdit".
metadata:
  tags: bryntum, editor, eventedit, taskedit, popup, scheduler, gantt
---

## Keep the built-in editor

**Default to Bryntum's built-in editor.** It already handles create/edit/delete, validation, and writes straight back to the stores. Do NOT build a custom dialog (a React/MUI modal, a framework dialog, hand-rolled HTML) unless the user explicitly asks for one.

To match a design-system host app (e.g. Material UI), switch the Bryntum **theme** (`material3-light` / `material3-dark`) so the built-in editor and bars match — see the `bryntum-theming` skill. That is not a reason to replace the editor.

---

## Which feature owns the editor

| Product | Editor feature |
|---|---|
| Scheduler | `eventEdit` |
| Scheduler Pro | `taskEdit` |
| Gantt | `taskEdit` |
| Task Board | `taskEdit` |
| Calendar | `eventEdit` |

---

## Customize via the feature's `items` config

Remove fields you don't want or add your own through the feature's `items`. Use **object notation** (not arrays) for built-in items — `name` binds a field to a data field.

**Built-in item refs differ by product** — a wrong ref is silently ignored (the field stays visible):

| Product | Feature | Built-in field refs (General tab / editor) |
|---|---|---|
| Gantt | `taskEdit` | `name`, `percentDone`, `effort`, `divider`, `startDate`, `endDate`, `duration` — **no `Field` suffix** |
| Scheduler Pro | `taskEdit` | `nameField`, `resourcesField`, `startDateField`, `endDateField`, `durationField`, `percentDoneField` |
| Scheduler, Calendar | `eventEdit` | `nameField`, `resourceField`, `startDateField`, `startTimeField`, `endDateField`, `endTimeField` |

Custom items can use any key (it becomes the new item's `ref`; `name` binds it to the data field). Only keys that change or remove a built-in item must match its ref exactly.

The `config-items` example on the Gantt `TaskEdit` API page is inherited from Scheduler Pro and uses `durationField` — in Gantt that key matches nothing and is ignored; use `duration`.

Tab refs (Gantt, Scheduler Pro): `generalTab`, `predecessorsTab`, `successorsTab`, `resourcesTab` (Gantt), `advancedTab`, `notesTab`. Fields on the Advanced tab use the `Field` suffix in both (e.g. `calendarField`, `constraintTypeField`). Verify refs for the installed version via MCP (`search_bryntum_docs`, "task editor fields").

Gantt:

```js
features: {
    taskEdit: {
        items: {
            generalTab: {
                items: {
                    percentDone : false,          // remove built-in fields (Gantt refs have no "Field" suffix)
                    effort      : false,
                    myField : { type: 'textfield', name: 'color', label: 'Color', weight: 710 } // add one
                }
            },
            predecessorsTab : false,              // remove whole tabs
            successorsTab   : false,
            advancedTab     : false
        }
    }
}
```

Scheduler Pro uses the same structure with `Field`-suffixed refs (`percentDoneField : false`, `durationField : false`). Scheduler and Calendar use `eventEdit` with flat `items` (no tabs), e.g. `eventEdit: { items: { resourceField: false } }`.

Run-time tweaks: listen to `beforeTaskEditShow` (or `beforeEventEditShow`) and adjust `editor.widgetMap.<ref>`.

---

## Replacing with a fully custom dialog (only when explicitly asked)

Keep the feature **ENABLED** and return `false` from `beforeEventEdit` / `beforeTaskEdit` to suppress the built-in popup.

**Do NOT set the feature to `false`** — that removes the hook entirely (silent no-op). Write changes back via `eventRecord.set({...})` so the stores stay in sync.

Framework-neutral example (a native `<dialog>`; the same contract applies to a React/Vue/Angular dialog component):

```js
let editingRecord = null;

const scheduler = new Scheduler({
    listeners : {
        beforeEventEdit({ eventRecord }) {
            editingRecord = eventRecord;

            // Fill and show your own dialog
            nameField.value  = eventRecord.name;
            startField.value = DateHelper.format(eventRecord.startDate, 'YYYY-MM-DD');
            dialog.showModal();

            // Prevent built-in editor
            return false;
        }
    }
});

// When clicking save in the custom dialog
saveButton.addEventListener('click', () => {
    // Update record — keeps the stores in sync
    editingRecord.set({
        name      : nameField.value,
        startDate : DateHelper.parse(startField.value, 'YYYY-MM-DD')
    });
    dialog.close();
});
```

In a framework, show your dialog component from the listener and commit on save the same way — the `beforeEventEdit`/`beforeTaskEdit` → `record.set()` contract is identical.

**Lifecycle note:** when you return `false`, your app owns the record lifecycle — for a newly drag-created event, remove it from the store if the user cancels the dialog (the built-in editor would have done this).
