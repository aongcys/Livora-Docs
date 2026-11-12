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
groups
goals
dashboard
fortune
calories
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

GET    /groups/:id/ranking/today
GET    /dashboard
GET    /fortune/today
```

Do not expose database implementation details directly through API contracts.

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
