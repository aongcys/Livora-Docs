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
