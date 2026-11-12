# Database

General rules and cross-table reference. Table definitions are in the other files in this folder:

| File | Tables |
| --- | --- |
| [users-tasks.md](users-tasks.md) | users, tasks, task_occurrences (implemented) |
| [habits-reminders.md](habits-reminders.md) | habits, habit_checkins, streaks, reminders, reminder_deliveries |
| [goals.md](goals.md) | goals |
| [xp-ranking.md](xp-ranking.md) | xp_events, XP idempotency |
| [extras.md](extras.md) | daily_fortunes, calorie profile |

When the schema changes, update the relevant file here and `src/docs-db/schema.mmd` in Livora-server. Section numbers keep the numbering of the original single-file DATABASE.md.

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
    ├──────── daily_fortunes
    │
    └──────── calorie_profiles / weight history
```

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
  ├── daily_fortunes
  └── calorie_profiles
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
- Groups are the exception: members may read the group tasks of groups they belong to, but never another user's personal tasks/habits/goals

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
- Group ranking logic
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
group ranking
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

Level / current XP
→ stored on users, updated in the same transaction as xp_events

Group ranking
→ group tasks completed today
```

---

# 26. Future Database Extensions

Possible V2 tables:

```text
friends
friend_requests
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
the relevant file in database/
src/docs-db/schema.mmd (Livora-server)
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
10. Group ranking is derived from completed group tasks, not stored.
11. Protect user data with RLS.
12. Do not trust client-provided `user_id`.
13. Keep business rules in NestJS.
14. Add indexes based on actual query patterns.
15. Update this document whenever the schema changes.
