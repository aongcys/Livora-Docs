# AGENT.md

# Project: Daily Life Companion

## 1. Project Overview

This project is a personal daily-life management web application designed to help users organize, track, and improve their everyday life in one place.

The product combines:

- Daily planning
- Tasks
- Calendar
- Reminders
- Habits
- Daily check-ins
- XP / Score
- Streaks
- Global ranking
- Goals
- Dashboard
- Daily fortune
- Calorie / daily calorie target
- Focus timer

The product is not intended to be only a traditional Todo application.

The main product concept is:

> Plan your day → complete activities → build habits → earn XP → maintain streaks → progress toward goals → compare progress through ranking.

The application should feel useful first and playful second. Gamification should encourage consistency without making the application feel like a game that overwhelms the productivity features.

---

# 2. Technology Stack

## Frontend

- Next.js
- TypeScript
- React
- App Router
- Responsive Web UI

## Backend

- NestJS
- TypeScript
- REST API (global prefix `/api`, no version segment)
- Prisma ORM with Prisma Migrate
- Zod for validation
- Swagger (OpenAPI) at `/docs`, enabled only when `NODE_ENV=development`

## Database / Backend Services

- Supabase
- PostgreSQL
- Supabase Auth
- Supabase Storage if file/image storage is needed later

## Architecture

```text
Browser
   |
   v
Next.js Frontend
   |
   | REST API
   v
NestJS Backend
   |
   v
Supabase / PostgreSQL
```

The frontend should not contain business-critical database logic.

The backend should be responsible for:

- Business rules
- Validation
- Authorization
- XP calculation
- Streak calculation
- Ranking calculation
- Goal progress logic
- Reminder logic
- Cross-feature operations

The frontend is responsible for:

- Rendering UI
- User interactions
- Local UI state
- Form handling
- API requests
- Client-side presentation logic

## 2.1 Repositories and Local Setup

The project is split into three independent repositories so each can be managed separately:

| Repo | Local folder | Purpose | Dev port |
| --- | --- | --- | --- |
| Livora-client | `client/` | Next.js frontend (App Router, code under `src/`) | 3200 |
| Livora-server | `server/` | NestJS backend | 4200 |
| Livora-Docs | `docs/` | This documentation | - |

Tooling:

- pnpm only (enforced by `only-allow`), Node 24 (see `.nvmrc`)
- Environment variables are loaded with `dotenvx run --` from an uncommitted `.env`; each repo commits a `.env.example`
- Server environment variables are validated at startup with Zod. Every variable is required and has no default; the app refuses to start and names the missing or invalid variable
- Server env: `NODE_ENV`, `PORT`, `CORS_ORIGIN` (comma-separated origins), `DATABASE_URL` (Supabase transaction pooler, port 6543, used by the running app), `DIRECT_URL` (Supabase session pooler, port 5432, used only by the Prisma CLI)
- Server linting uses oxlint; client linting uses ESLint; both use Prettier

---

# 3. Product Principles

## 3.1 Daily-first

The main question the application should answer is:

> "What do I need to do today, and how am I progressing?"

The Today view and Dashboard are therefore core product surfaces.

## 3.2 Features should connect

Features should not behave like completely independent mini-apps.

Example:

```text
Goal
  ↓
Plan
  ↓
Task / Habit
  ↓
Complete
  ↓
XP
  ↓
Level / Ranking / Progress
```

## 3.3 Productivity first, gamification second

Gamification should support the user's productivity.

Do not add XP to every possible action just because it is possible.

Examples of valid XP sources:

- Completing a task
- Completing a scheduled habit
- Completing a goal/milestone
- Daily check-in

Avoid XP exploits such as repeatedly creating and completing fake tasks.

## 3.4 Keep V1 focused

Do not implement V2/V3 features unless explicitly requested.

Potential future features:

- Friends
- Groups
- Private leaderboards
- Challenges
- AI planning
- Google Calendar integration
- Social feed
- Food database
- Mobile application
- Discord integration

---

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

---

# 6. Habit

A Habit represents something the user wants to repeat consistently.

Examples:

- Exercise
- Read a book
- Drink water
- Study
- Sleep before a specific time

A Habit is different from a Task.

Task:

```text
Run on September 20
```

Habit:

```text
Run
Every Tuesday and Thursday
```

### Core features

- Create habit
- Edit habit
- Delete habit
- Set schedule
- Daily check-in
- Undo check-in where appropriate
- Habit history
- Streak
- Current streak
- Best streak
- Completion rate
- Calendar history
- Optional reminder

### Habit schedule

A habit may have:

```text
DAILY
WEEKLY
CUSTOM
```

The system should create or calculate occurrences without duplicating unnecessary permanent rows.

### Streak

The system should track:

- Current streak
- Best streak
- Last completed date

A streak should be based on scheduled occurrences, not simply consecutive calendar days.

Example:

A habit scheduled only Monday and Thursday should not lose its streak because Tuesday was not scheduled.

---

# 7. XP / Score

XP is the gamification system.

Recommended name in the product can be either:

- XP
- Score

Internally, prefer a clear term such as `xp`.

### Example XP rules

```text
Complete task        +10 XP
Complete habit       +15 XP
Daily check-in        +5 XP
Complete milestone   +25 XP
```

Exact values are configurable and should not be scattered throughout the codebase.

Use a centralized XP rule configuration/service.

### XP requirements

XP events should be recorded so the system can explain where XP came from.

Example:

```text
Task completed
+10 XP
```

This also helps prevent duplicate XP.

### Anti-abuse requirement

Completing the same task multiple times must not repeatedly award XP.

XP awarding should be idempotent.

---

# 8. Level

Level is derived from XP.

Example:

```text
Level 1
0 - 99 XP

Level 2
100 - 249 XP

Level 3
250 - 499 XP
```

The exact formula can be changed later.

Do not permanently duplicate derived values unless there is a performance reason.

---

# 9. Ranking

V1 uses a global ranking.

Do NOT implement groups or friends in V1 unless explicitly requested.

### V1 ranking

- Global leaderboard
- Weekly ranking
- Current user's rank
- XP earned during the ranking period

Example:

```text
Rank    User        XP

1       Alex       1240
2       Mark       1180
3       John       1120
12      Current     980
```

### Ranking period

V1 should primarily use weekly ranking.

The ranking period should be explicit, for example:

```text
week_start
week_end
```

Do not rank users using lifetime XP for the main competitive leaderboard because older accounts would have a permanent advantage.

### Future

V2 may add:

- Friends
- Groups
- Group leaderboard
- Challenges
- Private competitions

---

# 10. Goals

Goals represent larger objectives.

Examples:

```text
Run 100 KM
Read 10 books
Finish project
Study 20 hours
```

### Core features

- Create goal
- Edit goal
- Delete goal
- Set start date
- Set deadline
- Set target
- Track current progress
- View progress percentage
- Goal status

### Goal status

```text
NOT_STARTED
IN_PROGRESS
COMPLETED
FAILED
CANCELLED
```

### Goal progress

A goal should support a measurable target where possible.

Example:

```text
Goal:
Run 100 KM

Current:
72 KM

Target:
100 KM

Progress:
72%
```

Recommended fields:

```text
target_value
current_value
unit
```

Do not force every future goal to use the same unit.

---

# 11. Dashboard

The Dashboard is the main page after login.

Its purpose is to provide a quick overview of the user's day.

The dashboard should answer:

> What do I need to do today?
> How am I progressing?
> What should I pay attention to?

### Recommended sections

#### Header

- Greeting
- Current streak
- Today's XP

#### Today's Tasks

- Upcoming task
- Completed task
- Task status
- Priority

#### Today's Habits

- Habit list
- Check-in action
- Current streak

#### Goals

- Active goals
- Progress
- Deadline/status

#### Ranking

- Current weekly rank
- XP this week

#### Daily Fortune

A small entertainment widget.

#### Optional widgets

- Calorie status
- Focus timer
- Daily progress percentage

The Dashboard should be customizable later, but V1 can use a fixed layout.

---

# 12. Daily Fortune

Daily Fortune is an entertainment feature.

It can provide randomized content such as:

- Overall luck
- Work
- Money
- Health
- Love
- Lucky number
- Lucky color
- Daily message

The system should treat this as entertainment/random content, not factual prediction.

### Important

The user should receive one daily result rather than a completely new result every refresh.

Example:

```text
User opens on Sep 20
→ generate/store today's fortune

User refreshes
→ same fortune

Sep 21
→ new fortune
```

This prevents refresh abuse and makes the feature feel like a daily event.

---

# 13. Calorie

V1 should keep calorie functionality lightweight.

### Initial scope

Calculate:

- BMR
- TDEE
- Daily calorie target

Input:

- Age
- Sex
- Height
- Weight
- Activity level
- Goal

Example:

```text
BMR
TDEE
Target Calories
```

### Calorie status

The daily calorie page may show:

```text
UNDER
ON_TARGET
OVER
```

Example:

```text
Target: 2000 kcal
Consumed: 2150 kcal

Status: OVER
```

Do not build a large food database in V1.

---

# 14. Focus Time

Focus Time is a lightweight productivity timer.

Example:

```text
Task: Study Machine Learning

25:00

[ Start ]
```

Possible V1 timer:

- Start
- Pause
- Resume
- Complete
- Reset

After a completed focus session, it may award XP.

The focus session should optionally link to a Task.

---

# 15. Authentication

Authentication is required to use the application.

Recommended:

- Supabase Auth
- Email/password
- OAuth can be added later

## 15.1 Who can do what

There are two kinds of visitors:

- **Guest (not logged in):** can only see the public landing page (hero section explaining what Livora is and its features). Guests cannot use any feature.
- **Logged-in user:** has a `role` of `USER` or `ADMIN`. Both are the same kind of account and log in the same way; the only difference is the role stored on the user record.

Whether someone is logged in is determined by verifying their token on the backend.

`ADMIN` differs from `USER` by having a user-management menu in the sidebar. Everything else is identical.

Every user-owned entity must be associated with a user.

Never trust `user_id` supplied by the client.

The backend must determine the authenticated user from the verified authentication context.

---

# 16. Authorization

Every user-owned resource must be protected.

Examples:

A user must not be able to:

- Read another user's private tasks
- Modify another user's habits
- Delete another user's goals
- Modify another user's XP
- Change another user's ranking data

Supabase Row Level Security should be considered an additional protection layer.

The NestJS API must also enforce authorization.

## 16.1 Roles

Hiding the admin menu in the frontend is only a UX convenience. The backend must always check the caller's role (for example with a Guard) on every admin-only endpoint. The frontend reads the role only to decide which menu items to show.

---

# 17. API Design

Use REST APIs.

All routes live under the `/api` prefix and there is no version segment (no `/v1`). Paths in the examples below are relative to `/api`. Interactive documentation is served at `/docs` in development only.

Recommended module structure:

```text
auth
users
tasks
calendar
reminders
habits
xp
levels
ranking
goals
dashboard
fortune
calories
focus
```

Example endpoints:

```text
GET    /tasks
POST   /tasks
GET    /tasks/:id
PATCH  /tasks/:id
DELETE /tasks/:id

POST   /tasks/:id/complete
POST   /tasks/:id/cancel
POST   /tasks/:id/skip

GET    /habits
POST   /habits
PATCH  /habits/:id
DELETE /habits/:id

POST   /habits/:id/check-in

GET    /goals
POST   /goals
PATCH  /goals/:id
DELETE /goals/:id

GET    /ranking/weekly
GET    /dashboard
GET    /fortune/today
POST   /focus/sessions
```

Do not expose database implementation details directly through API contracts.

---

# 18. Frontend Structure

Recommended Next.js structure:

```text
src/
├── app/
│   ├── (auth)/
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   ├── planner/
│   │   ├── calendar/
│   │   ├── habits/
│   │   ├── goals/
│   │   ├── ranking/
│   │   ├── calorie/
│   │   └── focus/
│   └── ...
│
├── components/
│   ├── ui/
│   ├── planner/
│   ├── habit/
│   ├── goal/
│   ├── dashboard/
│   ├── ranking/
│   └── ...
│
├── lib/
│   ├── api/
│   ├── auth/
│   ├── utils/
│   └── validation/
│
└── types/
```

The exact folder structure may change to match the existing project.

---

# 19. Backend Structure

Recommended NestJS modules:

```text
src/
├── auth/
├── users/
├── tasks/
├── reminders/
├── habits/
├── xp/
├── ranking/
├── goals/
├── dashboard/
├── fortune/
├── calories/
├── focus/
├── common/        (errors, filters, pipes)
├── config/        (environment schema and validation)
├── prisma/        (Prisma service and module)
├── health/        (GET /api/health database check)
└── main.ts
```

Each module should generally contain:

```text
controller
service
dto
entity/model types where appropriate
```

Business logic belongs in services, not controllers.

Controllers should remain thin.

---

# 20. Validation

Validate all external input.

Recommended:

- Zod schemas on both sides: the server validates every request (a `ZodValidationPipe`), and the client validates forms before sending
- Strong TypeScript types
- Each repo owns its own schemas (they are separate repositories, so schemas are not shared as code). The frontend calls the backend through a service layer
- Validate dates and times
- Validate enum values
- Validate numeric ranges

Never rely only on frontend validation.

---

# 21. Date and Time Rules

The application is daily/time-based, so date handling must be explicit.

Do not blindly use server local time.

Store timestamps consistently and convert to the user's timezone for display.

The user's timezone should be part of their profile/settings.

Important cases:

- Recurring habits
- Daily fortune
- Weekly ranking
- Reminder scheduling
- Calendar display
- Day boundaries
- Daylight-saving changes if international users are supported later

For V1, the application may target a single primary timezone but should avoid hard-coding timezone logic into business rules.

---

# 22. Error Handling

API errors should use predictable responses.

Every error has the same JSON shape:

```json
{
  "statusCode": 404,
  "code": "NOT_FOUND",
  "message": "Task not found",
  "details": []
}
```

- `code` is a stable machine-readable identifier (for example `VALIDATION_FAILED`, `NOT_FOUND`, `DATABASE_UNAVAILABLE`)
- `message` explains the cause in plain language
- `details` is optional; for validation errors it lists `{ field, message }` per invalid field
- Unexpected errors return `INTERNAL_ERROR` with a generic message; the real cause is logged on the server only

Frontend should display human-readable messages.

Examples:

```text
Task could not be created.
Reminder time is invalid.
You are not authorized to modify this task.
Goal was not found.
```

Do not expose stack traces or database errors to users.

---

# 23. Loading / Empty / Error States

Every major feature must consider:

### Loading

```text
Loading today's tasks...
```

### Empty

```text
No tasks today.
Create your first task.
```

### Error

```text
Something went wrong.
Try again.
```

### Success

```text
Task completed!
+10 XP
```

Do not design only the successful state.

---

# 24. UX Guidelines

The product should feel:

- Clean
- Friendly
- Lightweight
- Motivating
- Fast
- Easy to understand

Avoid making every screen look like a dashboard.

Use focused layouts for focused tasks.

Examples:

- Planner → information-dense
- Habit → visual/streak-oriented
- Ranking → competitive/table-oriented
- Dashboard → overview
- Focus → minimal/distraction-free

---

# 25. Gamification UX

Gamification should feel rewarding but not intrusive.

Good:

```text
Task completed ✓
+10 XP
```

Avoid:

- Excessive animations
- Constant popups
- Too many badges
- Forced social interaction

Users should still be able to use the product as a productivity tool.

---

# 26. Data Integrity Rules

Important business rules:

1. A task belongs to exactly one user.
2. A habit belongs to exactly one user.
3. A goal belongs to exactly one user.
4. XP events belong to exactly one user.
5. A user cannot award XP to themselves manually through a public API.
6. Completing the same action must not duplicate XP.
7. A skipped recurring task should not be treated as completed.
8. Habit streaks depend on the habit schedule.
9. Daily fortune should be stable for the same user/date.
10. Ranking should use the defined ranking period.
11. Users cannot access another user's private data.
12. Deleted entities should not leave broken references.

---

# 27. Suggested Development Order

Implement in this order:

## Phase 1

Authentication

↓

## Phase 2

Tasks / Planner

↓

## Phase 3

Calendar

↓

## Phase 4

Habits / Check-in

↓

## Phase 5

XP / Score

↓

## Phase 6

Goals

↓

## Phase 7

Ranking

↓

## Phase 8

Dashboard

↓

## Phase 9

Reminder

↓

## Phase 10

Fortune / Calorie / Focus

This order allows the core data flow to be tested before adding secondary features.

---

# 28. Definition of Done

A feature is not complete merely because the API works.

Each feature should include:

- Database model
- API
- Validation
- Authorization
- Frontend UI
- Loading state
- Empty state
- Error state
- Success state
- Responsive behavior
- Basic edge-case handling
- Tests where appropriate

---

# 29. Development Rules for Agents

When modifying this project:

1. Read this AGENT.md before making architectural changes.
2. Check existing code before creating new abstractions.
3. Reuse existing components and utilities.
4. Do not duplicate business logic between frontend and backend.
5. Keep controllers thin.
6. Keep business rules in backend services.
7. Do not bypass authentication or authorization.
8. Do not directly mutate Supabase data from the frontend for protected business operations unless the architecture explicitly allows it.
9. Use TypeScript strictly.
10. Avoid `any` unless there is a documented reason.
11. Do not silently change database schema.
12. Database migrations must be explicit.
13. Do not introduce V2 features into V1 without approval.
14. Do not create unnecessary dependencies.
15. Prefer simple solutions over premature abstractions.
16. Preserve backward compatibility when modifying API contracts.
17. Handle loading, empty, error, and success states.
18. Keep business rules testable.
19. Never trust client-provided user identity.
20. When a change affects multiple modules, update the relevant documentation.

---

# 30. Database Documentation

The database design is documented separately in:

```text
DATABASE.md
```

`DATABASE.md` defines:

- Tables
- Columns
- Relationships
- Enums
- Indexes
- Constraints
- RLS considerations
- XP event design
- Recurrence design
- Ranking design

When database structure changes, update `DATABASE.md`.

---

# 31. Commit Convention

```text
<type>: <message>
```

- Lowercase English, no trailing period
- `create: ...` for something new, `fix: ...` for a fix, `setup: ...` for project setup
- Everything else uses `feat: <what was done>`
- The first commit of each repository is `setup: project client`, `setup: project server` and `setup: project docs`
- Default branch is `main`
