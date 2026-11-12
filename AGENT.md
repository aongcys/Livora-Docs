# AGENT.md

Core project context and hard rules. Detailed specs are split into the files below so only what a task needs has to be read.

## Where to read for which task

| Task | Read |
| --- | --- |
| Full functional requirements (Thai FRS, all features) | [frs/Livora-FRS-TH.md](frs/Livora-FRS-TH.md) |
| Sign up, login, profile, admin | [features/auth.md](features/auth.md), [conventions/auth.md](conventions/auth.md), [database/users-tasks.md](database/users-tasks.md) |
| Tasks, recurrence, task history | [features/planner.md](features/planner.md), [database/users-tasks.md](database/users-tasks.md) |
| Calendar, holidays | [features/calendar.md](features/calendar.md) |
| Reminders, notifications | [features/reminders.md](features/reminders.md), [database/habits-reminders.md](database/habits-reminders.md) |
| Habits, check-ins, streaks | [features/habits.md](features/habits.md), [database/habits-reminders.md](database/habits-reminders.md) |
| XP, level | [features/xp-level.md](features/xp-level.md), [database/xp-ranking.md](database/xp-ranking.md) |
| Groups, group tasks, group ranking | [features/groups.md](features/groups.md) |
| Goals | [features/goals.md](features/goals.md), [database/goals.md](database/goals.md) |
| Dashboard | [features/dashboard.md](features/dashboard.md) |
| Fortune, calorie | [features/extras.md](features/extras.md), [database/extras.md](database/extras.md) |
| Auth, roles, authorization | [conventions/auth.md](conventions/auth.md), [database/users-tasks.md](database/users-tasks.md) |
| API routes, validation, error format | [conventions/api-and-errors.md](conventions/api-and-errors.md) |
| Folder structure (client/server) | [conventions/structure.md](conventions/structure.md) |
| Dates, timezones, day boundaries | [conventions/datetime.md](conventions/datetime.md) |
| UI states, UX, gamification UX, Definition of Done | [conventions/ux-and-done.md](conventions/ux-and-done.md) |
| Any schema change or migration | [database/README.md](database/README.md) (rules, indexes, FKs, migrations) and the table file it touches |

Section numbers (e.g. "section 15") in the files below keep the numbering of the original single-file AGENT.md.

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
- Groups (group tasks and in-group ranking)
- Goals
- Dashboard
- Daily fortune
- Calorie (weight and daily calories)

The product is not intended to be only a traditional Todo application.

The main product concept is:

> Plan your day → complete activities → build habits → earn XP → maintain streaks → progress toward goals → compare progress with your group.

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

- Supabase (PostgreSQL hosting only, not used for auth)
- PostgreSQL
- Custom authentication: the API owns sign-up/login and issues its own token (see [conventions/auth.md](conventions/auth.md))
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
- Group ranking calculation
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
Level / Group ranking / Progress
```

## 3.3 Productivity first, gamification second

Gamification should support the user's productivity.

Do not add XP to every possible action just because it is possible.

Examples of valid XP sources:

- Completing a task
- Completing a scheduled habit
- Completing a goal
- Daily check-in

Avoid XP exploits such as repeatedly creating and completing fake tasks.

## 3.4 Keep V1 focused

Do not implement V2/V3 features unless explicitly requested.

Potential future features:

- Friends
- Global leaderboard
- Challenges
- AI planning
- Google Calendar integration
- Social feed
- Food database
- Mobile application
- Discord integration

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
10. Group ranking counts group tasks completed today and resets daily at midnight Thai time.
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

Groups

↓

## Phase 8

Dashboard

↓

## Phase 9

Reminder

↓

## Phase 10

Fortune / Calorie

This order allows the core data flow to be tested before adding secondary features.

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

The database design lives in the [database/](database/README.md) folder: `README.md` holds the general rules, high-level relationships, indexes, foreign keys, migration rules and derived-data rules; each other file documents the tables of one area (tables, columns, enums, constraints).

When database structure changes, update the relevant file in `database/` and, in Livora-server, `src/docs-db/schema.mmd`.

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
