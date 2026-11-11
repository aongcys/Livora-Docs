# 17. Daily Fortune

```text
daily_fortunes
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id
fortune_date        date

overall_rating      integer
work_rating         integer nullable
money_rating        integer nullable
love_rating         integer nullable
health_rating       integer nullable

message             text
lucky_number        integer nullable
lucky_color         text nullable

created_at          timestamptz
```

Recommended unique constraint:

```text
(user_id, fortune_date)
```

This ensures a user receives one stable fortune per day.

---

# 18. Calorie Profile

User's calculation inputs.

```text
calorie_profiles
------------------------------
id                  uuid PK
user_id             uuid UNIQUE

age                 integer
sex                 calorie_sex
height_cm           numeric
weight_kg           numeric
activity_level      calorie_activity_level
goal                calorie_goal

created_at          timestamptz
updated_at          timestamptz
```

Enums:

```text
calorie_sex
MALE
FEMALE
OTHER
```

```text
calorie_activity_level
SEDENTARY
LIGHT
MODERATE
ACTIVE
VERY_ACTIVE
```

```text
calorie_goal
MAINTAIN
LOSE
GAIN
```

The exact calorie formula belongs in backend business logic.

---

# 19. Daily Calorie Logs

If users eventually record daily calories:

```text
calorie_daily_logs
------------------------------
id                  uuid PK
user_id             uuid FK
log_date            date

target_calories     numeric
consumed_calories   numeric

status              calorie_status

created_at          timestamptz
updated_at          timestamptz
```

Enum:

```text
UNDER
ON_TARGET
OVER
```

V1 does not require a full food database.

---

# 20. Focus Sessions

```text
focus_sessions
------------------------------
id                  uuid PK
user_id             uuid FK

task_id             uuid nullable FK -> tasks.id

started_at          timestamptz
ended_at            timestamptz nullable

planned_seconds     integer
actual_seconds      integer nullable

status              focus_status

created_at          timestamptz
```

Enum:

```text
RUNNING
PAUSED
COMPLETED
CANCELLED
```

A completed session may generate an XP event.
