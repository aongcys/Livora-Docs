# 4. V1 Feature Scope

## 4.1 Daily Planner

The Daily Planner is one of the primary features.

### Core features

- Create task
- Read task
- Update task
- Delete task
- Drag and drop task
- View today's tasks
- View upcoming tasks
- View all tasks
- Mark task as done
- Cancel task
- Skip recurring task occurrence
- Set task priority
- View task details
- Calendar view
- Task categories
- All-day tasks
- Recurring tasks

### Task information

A task may contain:

- Name
- Description
- Date
- Start time
- End time
- Priority
- Category
- Status
- All-day flag
- Repeat configuration
- Created timestamp
- Updated timestamp

Do not store a separate `day` field if it can be derived from `date`.

Example:

```text
date = 2026-09-20
day = Sunday
```

The weekday should normally be calculated from the date.

### Task status

Recommended initial statuses:

```text
TODO
IN_PROGRESS
DONE
CANCELLED
SKIPPED
```

### Priority

Recommended:

```text
LOW
MEDIUM
HIGH
```

Avoid overly complex priority systems in V1.

### Recurrence

Recurrence is separate from `all_day`.

Examples:

```text
All-day:
Doctor appointment

Recurring:
Read 30 minutes every day
```

Recommended recurrence types:

```text
NONE
DAILY
WEEKLY
MONTHLY
CUSTOM
```

V1 may initially implement NONE, DAILY, and WEEKLY and extend later if necessary.

---

# 5. Reminder

Reminder allows users to receive notifications before a task or habit occurs.

### Core features

- Enable / disable reminder
- Select target
- Set reminder amount
- Select reminder unit

Examples:

```text
15 minutes before
30 minutes before
1 hour before
2 hours before
```

The reminder system should be designed so that it can target more than only tasks.

Possible targets:

```text
TASK
HABIT
GOAL
```

### Important design principle

Do not hard-code reminders directly inside the Task model.

Use a generic reminder relationship:

```text
Reminder
  -> target_type
  -> target_id
```

This allows reminders to expand later.

### Notification states

A reminder may eventually have:

```text
PENDING
SENT
DISMISSED
FAILED
```

The exact notification infrastructure may be implemented later.
