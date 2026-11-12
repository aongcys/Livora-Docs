# Goals

A goal is a larger measurable objective, e.g. run 100 km.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-G1 | Create a goal: name, target number + unit, start date and deadline (both optional), linked habit (optional) |
| FR-G2 | Edit / delete a goal (confirm first). Deleting is also how a user cancels a goal |
| FR-G3 | Log progress manually (e.g. +5 km) |
| FR-G4 | Link a habit from either the goal form or the habit form; that habit's check-in amounts are added to the goal automatically |
| FR-G5 | Goal list and detail pages: progress (72 / 100 km · 72%), days left, progress log |
| FR-G6 | On reaching the target, a popup lets the user "raise target" or "finish goal" |
| FR-G7 | After the deadline passes (`FAILED`) the user can extend the deadline |

## Status (automatic)

```text
NOT_STARTED  new goal
IN_PROGRESS  first progress logged
COMPLETED    target reached and the user pressed "finish"
FAILED       deadline passed before the target; extending the deadline returns it to IN_PROGRESS
```

There is no `CANCELLED` status.

## Business rules

- Only one goal type: a numeric target + unit. No milestones and no task linking.
- Goal ↔ habit link is 1 to 1 and counted by amount (goal 140 glasses, today's check-in 5 → goal +5).
- Only count-type habits can be linked. Only check-ins after the link was made count.
- +25 XP once, when `COMPLETED`, and only if finished at least 1 day after creation.
- "Raise target" gives no XP; XP is only given when the goal is actually finished.
- Deleting a goal keeps the linked habit, just unlinked.

## Data needed (for DB design)

- `goals` (name, target, unit, current value, start date, deadline, status, linked habit).
- Goal progress log (amount, source manual or habit, time).
