# Livora Docs

Specification documents for **Livora** (Daily Life Companion).

| File | Purpose |
| --- | --- |
| [AGENT.md](AGENT.md) | Product scope, architecture, business rules and development rules |
| [DATABASE.md](DATABASE.md) | Database design (tables, enums, indexes, RLS, XP and ranking design) |

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
