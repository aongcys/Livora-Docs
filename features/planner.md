# Planner (Tasks)

## Pages

- **Today**: today's tasks.
- **Upcoming**: the next 7 days.
- **All**: every task, including tasks with no date.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-T1 | Create a task: name (required), description, date, start/end time, priority, category, repeat |
| FR-T2 | View details / edit / delete a task (hard delete, confirm first) |
| FR-T3 | Mark `DONE`, mark `CANCELLED`, revert to not finished. The user may optionally set `IN_PROGRESS` |
| FR-T4 | `SKIPPED` is only for a single day of a recurring task |
| FR-T5 | Drag and drop: on Today, reorder within the day; on All, drag to another date to move it |
| FR-T6 | Manage own categories: create, rename, change color, delete |
| FR-T7 | Filter by category, priority and status |
| FR-T8 | Task History: finished tasks (`DONE` / `CANCELLED` / `SKIPPED`). Today shows today's history; All shows everything with a 1 / 7 / 30 days / all filter, newest first. Status can be reverted from History |

## Flows

- **Create:** press add → enter a name (everything else optional) → save → appears on the page matching its date (no date = All only).
- **Complete:** tick the checkbox → `DONE` → tick again to revert.
- **Recurring task:** done / skip / cancel applies to that one day only.
- **Edit a recurring task:** applies from today onward; past days are unchanged.

## Statuses

```text
TODO         default for a new task
IN_PROGRESS  optional, set by the user
DONE
CANCELLED
SKIPPED      recurring tasks only, per day
```

## Business rules

- No start/end time = all-day task. If a time is given, both start and end are required and end must be after start.
- A recurring task needs a start date.
- V1 repeat types: `NONE`, `DAILY`, `WEEKLY`. Weekly can repeat on several chosen weekdays (e.g. Mon / Wed / Fri).
- Repeat end date is optional; without it the task repeats forever.
- An unfinished task past its date shows an "overdue" label; its status does not change automatically.
- Deleting a category keeps its tasks; they simply have no category.
- Default priority is `MEDIUM` (`LOW` / `MEDIUM` / `HIGH`).
- Do not store a separate weekday field; derive it from the date.
- Users only see their own tasks. Group tasks are described in [groups.md](groups.md).
- XP for completing tasks is described in [xp-level.md](xp-level.md).

## Data needed (for DB design)

- `categories` table (name, color, owner) and a category on `tasks`.
- Repeat weekdays and repeat end date on `tasks`.
- Sort order within a day (for drag and drop).
- Finished-at time for non-recurring tasks (for History filtering).
- `IN_PROGRESS` added to `task_occurrence_status`.

## Later (not V1)

- `MONTHLY` and `CUSTOM` repeat
