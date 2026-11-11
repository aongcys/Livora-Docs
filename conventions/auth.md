# 15. Authentication

Authentication is required to use the application.

Implementation: custom, not Supabase Auth.

- The API owns email/password sign-up and login; credentials live in the `users` table
- Passwords are hashed (bcrypt/argon2) in `password_hash`, never stored or logged in plain text
- The API issues its own token (JWT) on login; the backend verifies it on every request
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

Not enforced by Supabase Row Level Security — that relies on Supabase Auth, which this project doesn't use. The NestJS API is the only authorization layer; every endpoint must check ownership against the authenticated user from the verified token. See [database/README.md](../database/README.md) section 23.

## 16.1 Roles

Hiding the admin menu in the frontend is only a UX convenience. The backend must always check the caller's role (for example with a Guard) on every admin-only endpoint. The frontend reads the role only to decide which menu items to show.
