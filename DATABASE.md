# DATABASE.md

# Database Design: Daily Life Companion

## 1. Database

Database:

- PostgreSQL
- Hosted by Supabase

Authentication:

- Custom (not Supabase Auth): email + password, hashed with bcrypt/argon2, stored in `users`
- The API issues and verifies its own JWT

The application database should use UUID primary keys where practical.

Recommended:

```sql
id uuid primary key default gen_random_uuid()
```

All user-owned tables should reference the authenticated user.

Access and tooling:

- Prisma ORM (version 7) is the data access layer and Prisma Migrate manages the schema
- The running app connects through the Supabase **transaction pooler** (`DATABASE_URL`, port 6543)
- Prisma CLI commands (migrate, studio) use the Supabase **session pooler** (`DIRECT_URL`, port 5432), configured in `prisma.config.ts`

Status: as of 2026-09-21, `users`, `tasks` and `task_occurrences` are implemented and migrated (see their sections below for the actual shape). Everything else below is still the plan.

---

# 2. High-Level Relationship

```text
users
    |
    ├──────── tasks
    │            |
    │            ├──── task_occurrences
    │            └──── reminders
    │
    ├──────── habits
    │            |
    │            ├──── habit_occurrences / check-ins
    │            └──── reminders
    │
    ├──────── goals
    │
    ├──────── xp_events
    │
    ├──────── focus_sessions
    │
    ├──────── daily_fortunes
    │
    └──────── calorie_logs / calorie_targets
```

---

# 3. users

Application-level user record. Authentication is custom (not Supabase Auth), so this table also owns the account's credentials.

```text
users
------------------------------
id              uuid PK, default gen_random_uuid()
email           text UNIQUE (partial: WHERE deleted_at IS NULL)
username        text UNIQUE (partial: WHERE deleted_at IS NULL), display only, not used to log in
password_hash   text
avatar_url      text nullable
gender          gender
timezone        text, default 'Asia/Bangkok'
role            user_role, default USER
created_at      timestamptz
updated_at      timestamptz
deleted_at      timestamptz nullable (soft delete)
```

## Enums

```text
user_role
USER
ADMIN
```

```text
gender
MALE
FEMALE
```

`ADMIN` and `USER` are the same kind of account. The role only controls what the backend allows (for example the user-management endpoints) and which menu items the frontend shows. Visitors who are not logged in have no row here. `role` defaults to `USER` and must never be set from a client-supplied value; only the backend, acting for an admin, may change it.

`email`/`username` uniqueness is a **partial** unique index (`WHERE deleted_at IS NULL`), so a soft-deleted account's email/username can be reused by a new account. Prisma cannot express a partial unique index declaratively, so it is added as raw SQL inside the Prisma migration.

`password_hash` must be a proper hash (bcrypt/argon2) — never store or log a plain-text password. `gender` is required at registration; it is used later for the first calorie calculation, so the user isn't asked twice.

---

# 4. tasks

Represents user-created tasks/plans.

```text
tasks
------------------------------
id                  uuid PK, default gen_random_uuid()
user_id             uuid FK -> users.id, ON DELETE CASCADE
name                text
description         text nullable

date                date nullable
start_time          time nullable
end_time            time nullable

priority            task_priority
status              task_status
repeat_type         repeat_type

created_at          timestamptz
updated_at          timestamptz
```

## Enums

```text
task_priority
LOW
MEDIUM
HIGH
```

```text
task_status
TODO
IN_PROGRESS
DONE
CANCELLED
SKIPPED
```

```text
repeat_type
NONE
DAILY
WEEKLY
MONTHLY
CUSTOM
```

There is no `is_all_day` column: a task with `start_time`/`end_time` both `null` is an all-day task by definition, so the flag would only duplicate that.

There is no `repeat_config` in V1: `WEEKLY` repeats on the same weekday as `date`, which needs no extra config. Add `repeat_config` (nullable jsonb) later, additively, once `CUSTOM`/`MONTHLY` need an explicit pattern (e.g. `{"days": ["MONDAY", "WEDNESDAY"]}`) that can't be inferred from `date` alone.

Indexes: `(user_id, date)`, `(user_id, status)`.

---

# 5. Task Completion

For simple non-recurring tasks (`repeat_type = NONE`), `tasks.status` is the source of truth.

For recurring tasks (`repeat_type != NONE`), the base task is a template/rule — its `status` is not overwritten per day. Per-day state is tracked separately in `task_occurrences`.

---

# 6. task_occurrences

Only for recurring tasks. A row exists **only** when a specific date was acted on (completed, cancelled or skipped); it is created lazily, not pre-generated for future dates. A date with no row is still `TODO`, derived from the parent task's `repeat_type` + `date`.

```text
task_occurrences
------------------------------
id                  uuid PK, default gen_random_uuid()
task_id             uuid FK -> tasks.id, ON DELETE CASCADE
occurrence_date     date
status              task_occurrence_status
completed_at        timestamptz nullable
created_at          timestamptz
updated_at          timestamptz
```

## Enum

```text
task_occurrence_status
DONE
CANCELLED
SKIPPED
```

Unique constraint (also serves as its index): `(task_id, occurrence_date)` — prevents duplicate occurrences.

---

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

---

# 12. goals

```text
goals
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id

name                text
description         text nullable

start_date          date
deadline            date nullable

target_value        numeric nullable
current_value       numeric nullable
unit                text nullable

status              goal_status

created_at          timestamptz
updated_at          timestamptz
```

Enum:

```text
goal_status
NOT_STARTED
IN_PROGRESS
COMPLETED
FAILED
CANCELLED
```

Progress can be calculated:

```text
progress = current_value / target_value
```

The backend should clamp display progress to 0–100%.

---

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

---

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

---

# 21. Indexes

Important indexes should include:

```text
tasks(user_id, date)
tasks(user_id, status)

task_occurrences(task_id, occurrence_date)
task_occurrences(occurrence_date)

habits(user_id, is_active)

habit_checkins(habit_id, checkin_date)
habit_checkins(user_id, checkin_date)

reminders(user_id, enabled)

goals(user_id, status)
goals(user_id, deadline)

xp_events(user_id, created_at)
xp_events(created_at)

daily_fortunes(user_id, fortune_date)

calorie_daily_logs(user_id, log_date)

focus_sessions(user_id, started_at)
```

Exact indexes should be adjusted after query patterns are known.

---

# 22. Foreign Keys

Recommended ownership relationships:

```text
users
  ├── tasks
  ├── habits
  ├── goals
  ├── reminders
  ├── xp_events
  ├── focus_sessions
  ├── daily_fortunes
  ├── calorie_profiles
  └── calorie_daily_logs
```

Child entities should use appropriate `ON DELETE` behavior.

For user-owned data, deleting the parent user should generally remove or archive dependent application data according to the product's account-deletion policy.

For normal entity deletion:

```text
Deleting a habit
→ habit check-ins should be deleted or archived
```

```text
Deleting a task
→ task occurrences and task reminders should be handled
```

Do not leave orphaned records.

---

# 23. Row Level Security

Not used. Supabase RLS's usual `auth.uid() = user_id` rule only works through Supabase Auth (Postgres reads the caller's JWT claim); this project uses custom auth, and the API connects to Postgres through a single pooled connection with no per-user JWT reaching the database. So RLS is not a working protection layer here.

Authorization is enforced only by the NestJS backend (guards + service-level checks against the authenticated user from the verified token), on every endpoint:

- Users can only SELECT/INSERT/UPDATE/DELETE their own tasks (and other owned rows)
- Users cannot directly access another user's private data
- Public ranking is the exception: users may read aggregated ranking information, but not another user's private task/habit/goal data

If per-row database-level enforcement is needed later, it would have to be custom (e.g. setting a session variable per request), not Supabase's built-in `auth.uid()`.

---

# 24. Backend vs Database Responsibilities

## Database

Responsible for:

- Persistence
- Foreign keys
- Unique constraints
- Basic integrity
- RLS
- Indexes

## NestJS

Responsible for:

- Business rules
- XP awarding
- Streak calculation
- Goal status transitions
- Reminder validation
- Ranking logic
- Recurrence logic
- Authorization
- API validation

Do not attempt to put the entire application's business logic into PostgreSQL triggers.

---

# 25. Derived Data

Prefer calculating these from source data:

```text
weekday
task completion percentage
habit streak
goal progress
total XP
level
weekly XP
ranking
```

Cache/materialize only when there is a demonstrated performance need.

Source-of-truth examples:

```text
XP
→ xp_events

Habit completion
→ habit_checkins

Goal progress
→ goal current_value / linked progress events if added

Ranking
→ xp_events within period
```

---

# 26. Future Database Extensions

Possible V2 tables:

```text
friends
friend_requests
groups
group_members
challenges
challenge_participants
achievements
user_achievements
notifications
calendar_integrations
external_calendar_events
```

Do not create these tables in V1 unless required.

---

# 27. Migration Rules

Every database schema change must be represented by a migration.

Migrations are created with Prisma Migrate (`prisma/migrations`, committed to the Livora-server repo). Things Prisma cannot model, such as partial unique indexes (see `users`), RLS policies and triggers, are written as raw SQL inside the same migration file. Never change the schema by hand in the Supabase dashboard.

Commands (run from `server/`): `pnpm db:migrate:dev` (create and apply during development), `pnpm db:migrate:deploy` (apply in deployed environments), `pnpm db:migrate:reset` (development only, destroys data), `pnpm db:studio`.

Do not manually modify production schema without a migration.

When changing:

- Table
- Column
- Enum
- Constraint
- Index
- RLS policy

Update:

```text
DATABASE.md
```

and the corresponding migration.

---

# 28. Important Database Rules

1. Use UUIDs for application entity IDs.
2. Every user-owned table must have an ownership relationship.
3. Use timestamps with timezone.
4. Use `date` for date-only concepts.
5. Use `time` for time-only values where appropriate.
6. Do not store derived weekday values unnecessarily.
7. Use unique constraints to prevent duplicate daily check-ins.
8. Make XP events idempotent.
9. Keep recurrence separate from all-day state.
10. Do not store lifetime ranking as the only ranking mechanism.
11. Protect user data with RLS.
12. Do not trust client-provided `user_id`.
13. Keep business rules in NestJS.
14. Add indexes based on actual query patterns.
15. Update this document whenever the schema changes.
