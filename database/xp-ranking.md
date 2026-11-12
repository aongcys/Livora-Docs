# 13. XP Events

Use an event ledger instead of only storing a mutable total XP value.

```text
xp_events
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id

event_type          xp_event_type
source_type         text
source_id           uuid nullable

amount              integer

metadata            jsonb nullable
created_at          timestamptz
```

Example:

```text
event_type = TASK_COMPLETED
source_type = TASK
source_id = task UUID
amount = 10
```

## Event types

```text
TASK_COMPLETED
HABIT_COMPLETED
DAILY_CHECKIN
GOAL_COMPLETED
```

The exact XP values should be defined by backend business rules.

---

# 14. XP Idempotency

XP must not be awarded repeatedly for the same source event.

Recommended unique logic:

```text
event_type + source_type + source_id + user_id
```

For events where the same source can legitimately award XP multiple times, use a more specific event key.

Example:

```text
habit_id + checkin_date
```

The backend should create XP events atomically with the completion action where possible.

---

# 15. Ranking

The global/weekly ranking was removed. Ranking now happens inside groups and is derived from group tasks completed today; see [features/groups.md](../features/groups.md).
