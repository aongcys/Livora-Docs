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
