# Reminders

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-M1 | Set a reminder inside the create / edit form of a task, group task or habit (1 item = 1 reminder) |
| FR-M2 | Lead time options: 15 min / 30 min / 1 h / 2 h / 1 day |
| FR-M3 | Enable / disable per item (default off) |
| FR-M4 | Two channels: in-app bell and browser notification |
| FR-M5 | Bell: unread count, notification list, click opens the item, mark read / dismiss |

## Business rules

- All-day tasks and habits without a time are reminded at 08:00 that day (user's timezone).
- Recurring tasks / habits are reminded on every due day.
- An item marked done before the reminder time is not reminded (a count habit is not reminded once its target is reached).
- Group task: each assignee sets their own reminder.
- Browser notifications need the user's permission; if denied, only the in-app bell is used.
- No snooze in V1.

## Design principle

Do not hard-code reminders inside the Task model. Use a generic target:

```text
Reminder
  -> target_type   (TASK / HABIT)
  -> target_id
```

Notification states may eventually be `PENDING`, `SENT`, `DISMISSED`, `FAILED`.

## Data needed (for DB design)

- `reminders` (owner, target type + id, lead time, enabled).
- `notifications` for the bell (message, read or not, link to the item).
- Browser push subscriptions per device.

## Later (not V1)

- Email reminders, snooze, multiple reminders per item
