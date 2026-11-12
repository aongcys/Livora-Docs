# Groups

Replaces the global ranking. Works like group work: members create group tasks, assign people responsible, mark them done, and everyone in the group sees the progress. Personal tasks are never shown in a group. XP stays one overall total per user.

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-R1 | Create a group (anyone) and get an invite code / link |
| FR-R2 | Join with the code / link; leave a group at any time |
| FR-R3 | Owner: rename, remove members, regenerate the invite code, delete the group |
| FR-R4 | Any member can create a group task and assign one or more members responsible |
| FR-R5 | Group page: group tasks with assignees and status (not done / done) |
| FR-R6 | Group ranking: number of group tasks completed today per member, reset daily |

## Business rules

- Max 20 members per group; a user can be in max 20 groups.
- Personal tasks are never visible in groups; groups only see group tasks.
- Group tasks have the same fields as normal tasks, including repeat.
- Group tasks appear on each assignee's Today and Calendar pages with the group name.
- Any assignee marking a task done completes it for everyone; every assignee gets +10 XP and +1 in today's ranking.
- Group task XP counts toward the 200 XP/day task cap.
- A group task can be edited / deleted by its creator, its assignees and the group owner.
- The ranking resets at midnight Thai time; on a tie, whoever reached the count first ranks higher.
- Leaving, being removed or the group being deleted removes that group's tasks from the user's pages. XP already earned is kept.
- If a leaving member shared a task with others, the task stays in the group without them.

## Data needed (for DB design)

- `groups` (name, owner, invite code).
- `group_members` (group, user, joined at).
- `tasks`: group reference (empty = personal task).
- Group task assignees (task, user), many per task.
- The daily ranking is derived from completed group tasks, not stored.
