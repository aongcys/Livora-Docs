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
GOAL_MILESTONE
FOCUS_SESSION_COMPLETED
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

Do not create a permanent ranking row for every user every time the leaderboard is viewed unless needed for performance.

Weekly ranking can be calculated from:

```text
xp_events
```

using the relevant date range.

Example:

```text
week_start <= created_at < week_end
```

Then:

```text
SUM(xp_events.amount)
GROUP BY user_id
ORDER BY SUM(amount) DESC
```

For high scale, a materialized/cached leaderboard can be added later.

---

# 16. Ranking Period

Ranking should use a consistent timezone and week boundary.

Recommended fields when storing cached periods:

```text
ranking_periods
------------------------------
id
period_type
start_at
end_at
created_at
```

V1 may not need this table if weekly ranking is calculated dynamically.
