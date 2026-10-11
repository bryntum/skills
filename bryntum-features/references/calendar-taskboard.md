# Calendar and TaskBoard features

## Calendar

- Views: `mode` + `modes`. `day`, `week`, `month`, `year`, `agenda` are present by default;
  `week` is the default `mode`. Add others via `modes : { list : true, resource : true }` (also
  `dayresource`, `dayagenda`, `monthagenda`, `monthgrid`, `yearplanner`); remove one with
  `modes : { year : null }`.
- Recurring events: `recurrenceRule` (RRULE string) + `exceptionDates`. This lives on the shared
  `TimeSpan` mixin, so Scheduler, Scheduler Pro and time ranges support it too.
- Hide weekends / first day of week: `nonWorkingDays`, `hideNonWorkingDays`, `weekStartDay`.
- Visible hours: `dayStartTime` / `dayEndTime`, or `workingTime : { fromHour, toHour, fromDay, toDay }`.
  This is a different mechanism from the Pro / Gantt `CalendarModel` intervals.
- Sidebar resource filter: `sidebar : { resourceFilter : { … } }`.
- Calendar has no dependencies.

## TaskBoard

- Swimlanes: `swimlaneField` + `swimlanes`, or `autoGenerateSwimlanes : true`.
- Card content: `headerItems` / `bodyItems` / `footerItems`, keyed by field name, with `type`
  naming the item type: `bodyItems : { prio : { type : 'text' }, tags : { type : 'tags', field : 'labels' } }`.
  Yours merge with the defaults (`headerItems.text`→`name`, `bodyItems.text`→`description`,
  `footerItems.resourceAvatars`); set one to `null` to remove it. Item types: `text`, `template`,
  `tags`, `progress`, `rating`, `image`, `chart`, `todoList`, `resourceAvatars`, `taskMenu`,
  `separator`, `collapse`, `jsx`.
- `tasksPerRow` (board, column or swimlane) is how many cards sit side by side — a layout
  setting, not a capacity limit.
- Filtering: `columnFilter` (honours `ColumnModel.filterable`) or `filterBar` (honours `searchable`).
- Pin a column to an edge: `locked : 'start' | 'end'`.

### WIP limits

There is no `wipLimit`, `maxTasks` or `capacity` config. If the user wants one, veto the drop
yourself and tell them it's a custom limit:

```javascript
listeners : {
    beforeTaskDrop({ targetColumn, taskRecords }) {
        if (targetColumn.id === 'doing' && targetColumn.tasks.length + taskRecords.length > 3) {
            return false;
        }
    }
}
```
