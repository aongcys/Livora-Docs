# Auth and User Accounts

Custom authentication owned by the API (not Supabase Auth). Security and authorization rules live in [conventions/auth.md](../conventions/auth.md).

## Who uses it

- **Guest** (not logged in): sees only the public landing page.
- **User**: role `USER`.
- **Admin**: role `ADMIN`, same account type plus a user-management menu.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-A1 | Sign up with email, username, password, gender and timezone |
| FR-A2 | Log in with email + password |
| FR-A3 | Log out |
| FR-A4 | View / edit profile: username, gender, timezone, UI language. Email cannot be changed |
| FR-A5 | Change password (requires the current password) |
| FR-A6 | Delete own account (requires the password, soft delete) |
| FR-A7 | Admin: list users, change role, suspend / unsuspend, delete a user |
| FR-A8 | Guests can only see the landing page |

## Flows

- **Sign up:** fill the form → pick a country (pick a city only when the country has several timezones; default Thailand) → validate → create account with role `USER` → log in automatically → Dashboard.
- **Log in:** email + password → success creates a 7-day session → Dashboard. Failure shows "email or password is incorrect" and never says which field was wrong.
- **Session expired:** redirect to login, then back to the page the user was on.
- **Delete account:** press delete → enter password → confirm → logged out immediately.

## Business rules

- Email and username are unique among non-deleted accounts.
- Password: at least 8 characters, containing letters and digits.
- A session lasts a fixed 7 days (no rolling renewal) and ends early on logout.
- Only an admin can change a role.
- An admin cannot demote or delete themselves, so there is always at least one admin.
- A suspended user cannot log in, and their existing sessions stop working immediately.
- Deleted and suspended accounts are not shown in groups or rankings.
- Defaults: UI language English (switchable to Thai), timezone `Asia/Bangkok`.
- The first admin is created by a seed script.
- Avatar in V1 is the first letter of the username.

## Data needed (for DB design)

- `users`: UI language (`en` / `th`), suspended status.
- New `sessions` table so logout, suspend and delete can revoke sessions server-side.

## Later (not V1)

- Email verification and forgot-password by email (a free tier such as Brevo is an option)
- Avatar upload
- OAuth
