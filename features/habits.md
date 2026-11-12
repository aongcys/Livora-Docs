# Habits

A habit is something the user wants to repeat consistently. Unlike a recurring task it has a streak, statistics and no end date.

```text
Task:  Run on September 20
Habit: Run every Tuesday and Thursday
```

## Habit types

- **Yes / No**: e.g. meditate.
- **Count**: a daily target + unit, e.g. drink water 8 glasses, push-ups 50 times.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-H1 | Create a habit: name, description, type, target + unit (count type), schedule (every day / chosen weekdays), color / icon, intended time (optional), linked goal (optional, see [goals.md](goals.md)) |
| FR-H2 | Edit / delete a habit (deleting also removes its history; confirm first) |
| FR-H3 | Check in for today: yes/no = one tap; count = a +1 button or typing a number |
| FR-H4 | Undo / change today's check-in |
| FR-H5 | Habit list page: every habit with its current streak and today's progress |
| FR-H6 | Habit detail page: current streak, best streak, 30-day success rate, monthly totals (e.g. "186 glasses this month"), and a monthly calendar showing target-reached days vs partial days |
| FR-H7 | App daily check-in: opening the app once a day earns XP (see [xp-level.md](xp-level.md)) |

## Business rules

- Check-ins are only for today. Past days are closed and cannot be edited.
- Count type:
  - more than 0 but below target → streak continues, amount added to totals, not counted as a target-reached day in the success rate
  - target reached → streak continues and counts as a target-reached day
  - 0 → streak breaks
- Streaks are based on scheduled days only; unscheduled days never break a streak.
- Missing one scheduled day resets the streak to 0.
- No pause or archive: not doing it means missing it.
- Changing a target or schedule applies from today; past days keep the old values.
- A new habit starts counting from the day it is created.
- Days are computed in the user's timezone.

## Data needed (for DB design)

- `habits` (type, target, unit, schedule weekdays, color / icon, intended time).
- Daily check-ins: one row per habit per day with the amount and that day's target.
- Streaks and totals are derived from check-ins, not stored (unless performance requires it later).
- Daily app-visit log (for FR-H7 and the Dashboard streak).
