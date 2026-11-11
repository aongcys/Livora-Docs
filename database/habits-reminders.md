# 7. habits

Represents a recurring behavior.

```text
habits
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id
name                text
description         text nullable
schedule_type       habit_schedule_type
schedule_config     jsonb nullable
is_active            boolean
created_at           timestamptz
updated_at           timestamptz
```

## Enum

```text
habit_schedule_type
DAILY
WEEKLY
CUSTOM
```

Example:

```json
{
  "days": ["TUESDAY", "THURSDAY"]
}
```

---

# 8. habit_checkins

Records completion of a habit for a date.

```text
habit_checkins
------------------------------
id                  uuid PK
habit_id            uuid FK -> habits.id
user_id             uuid FK -> users.id
checkin_date        date
checked_at          timestamptz
```

Recommended unique constraint:

```text
(habit_id, checkin_date)
```

This prevents duplicate check-ins.

The `user_id` can be retained for easier RLS/indexing, but backend logic must ensure it matches the habit owner.

---

# 9. Streaks

Do not necessarily store current streak and best streak as the source of truth.

They can be calculated from check-ins and schedules.

If performance later requires caching:

```text
habit_stats
------------------------------
habit_id
current_streak
best_streak
last_completed_date
updated_at
```

But this should be treated as derived/cache data.

Source of truth:

```text
habits
+
habit_checkins
```

---

# 10. reminders

Generic reminder model.

```text
reminders
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id

target_type         reminder_target_type
target_id           uuid

amount              integer
unit                reminder_unit

enabled             boolean

created_at          timestamptz
updated_at          timestamptz
```

## Enums

```text
reminder_target_type
TASK
HABIT
GOAL
```

```text
reminder_unit
MINUTE
HOUR
DAY
```

Example:

```text
target_type = TASK
target_id = task UUID
amount = 30
unit = MINUTE
```

---

# 11. reminder_deliveries

Optional table for actual notification delivery history.

```text
reminder_deliveries
------------------------------
id                  uuid PK
reminder_id         uuid FK
scheduled_at        timestamptz
sent_at             timestamptz nullable
status              reminder_delivery_status
error_message       text nullable
```

Enum:

```text
PENDING
SENT
FAILED
DISMISSED
```

This may be omitted during the first implementation if notifications are initially simple.
