# XP and Level

XP is the gamification layer. It should encourage consistency without overwhelming the productivity features.

## XP values

All values live in one central backend config, never scattered through the code.

| Source | XP |
| --- | --- |
| Task completed (every priority equal) | +10 |
| Habit target reached | +5 |
| Habit done but below target | +1 |
| App daily check-in (first app open of the day) | +2 |
| Goal completed | +25 |
| Habit streak bonus, per habit | 10 days +10, 20 days +12, 30 days +14 ... (+2 XP every further 10 days) |

## Level

- Level 1 → 2 needs 100 XP; each next level needs 10% more (110, 121, 133 ... rounded down).
- On level up the XP bar restarts; overflow carries over (95/100 + 10 = Level 2 with 5 XP).
- Level up shows a congratulation popup.
- A level never goes down. Undo can drop the bar to 0 at most.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-X1 | Award XP automatically according to the table above |
| FR-X2 | Show level and XP bar (e.g. Lv 3 · 45 / 121 XP) |
| FR-X3 | XP history page: source, time, amount |
| FR-X4 | Task and habit History show the XP earned per item |

## Business rules

- The same activity earns XP only once (idempotent).
- Reverting a task from `DONE` warns that the XP will be removed; on confirm the XP is deducted.
- XP from tasks (personal + group) is capped at 200 per day. Tasks can still be completed after the cap, without XP, with a message.
- `CANCELLED` / `SKIPPED` tasks earn no XP.
- A goal earns XP only if completed at least 1 day after it was created.
- A streak bonus can be earned again after a streak breaks and rebuilds.
- An admin cannot edit a user's XP, and there is no public API to award XP manually.
- XP is applied instantly in the same transaction as the completion.

## Data needed (for DB design)

- `users`: level and current XP in the bar, updated in the same transaction.
- `xp_events` ledger (who, source, amount, when), written in the same transaction; used for XP history, group activity and idempotency.

## Open

- What a level up unlocks for the user is not decided yet.
