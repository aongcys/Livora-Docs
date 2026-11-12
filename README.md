# Livora Docs

Specification documents for **Livora** (Daily Life Companion).

| Path | Purpose |
| --- | --- |
| [AGENT.md](AGENT.md) | Start here: project overview, stack, principles, hard rules, and an index of which file to read for which task |
| [frs/Livora-FRS-TH.md](frs/Livora-FRS-TH.md) | Functional Requirements Specification (Thai): every V1 feature with FR tables, user flows and business rules |
| [features/](features/) | Feature specs: auth, planner, calendar, reminders, habits, XP/level, goals, groups, dashboard, extras |
| [conventions/](conventions/) | Auth, API/errors/validation, folder structure, date/time, UX and Definition of Done |
| [database/](database/README.md) | Database rules plus per-area table definitions |

## Repositories

| Repo | Stack |
| --- | --- |
| [Livora-client](https://github.com/aongcys/Livora-client) | Next.js (App Router) |
| [Livora-server](https://github.com/aongcys/Livora-server) | NestJS + Prisma + Supabase |
| [Livora-Docs](https://github.com/aongcys/Livora-Docs) | This repo |

Recommended local layout, so all three can be read side by side:

```text
Livora/
├── client/   # Livora-client
├── server/   # Livora-server
└── docs/     # Livora-Docs
```

## Commit convention

```text
<type>: <message>
```

Lowercase English, no trailing period. Types: `create`, `fix`, `setup`; anything else is `feat: <what was done>`.
