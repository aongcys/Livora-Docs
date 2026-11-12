# Calendar

A separate page (not on the Dashboard). The system places tasks and habits on their days automatically; the user does not enter calendar items.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-C1 | Automatically place tasks and habits on days (recurring tasks on every repeat day, habits on scheduled days) |
| FR-C2 | Month view with navigation to other months and a Today button |
| FR-C3 | Click a day: open that day's list, add a task on that date, delete a task |
| FR-C4 | Drag a task to another day to move its date |
| FR-C5 | Show Thai public holidays and major festivals (holiday dates in red, hover shows the name) |
| FR-C6 | Admin: add / edit / delete holidays |

## UI

- A day cell shows about 3 items, then "+N more" which opens the day panel.
- Finished tasks / habits are faded and struck through.

## Business rules

- The week starts on Sunday (Thai calendar).
- Thai UI shows the Buddhist year (B.E.); English UI shows A.D.
- Dates are computed in the user's timezone.
- Tasks without a date are not shown.
- Moving or deleting a recurring task from the calendar asks first: "this day only" or "whole series".
- Deleting always asks for confirmation.
- Holidays: fixed-date ones (New Year, Songkran, ...) repeat yearly automatically; lunar ones (Makha Bucha, Visakha Bucha, Asanha Bucha, Buddhist Lent) are entered by an admin once a year.

## Data needed (for DB design)

- `holidays` table (Thai / English name, date, repeats yearly or not).
- No separate calendar table; the calendar is built from tasks and habits at display time.
- Moving one day of a recurring task needs a per-day override (e.g. a moved date on `task_occurrences`).

## Later (not V1)

- Google Calendar integration
