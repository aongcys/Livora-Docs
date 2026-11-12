# Dashboard

The main page after login. It answers one question: "What do I need to do today?" Only essentials; fixed layout in V1.

## Layout

```text
+----------------------------------------------------------------+
| Good evening, <name>     Lv 5 [#######-----] 80/146   🔥12  [🔮] |
+-----------------------------------------+----------------------+
| ! 2 overdue tasks              [Show]   | TODAY'S HABITS       |
| TODAY TIMELINE                          | [x] Meditate    8d   |
| All day                                 | [ ] Water 5/8   [+]  |
|   [ ] Pay electricity bill              | [ ] Push-ups 20/50[+]|
| 09:00  [x] Team meeting   (faded)       |                      |
| ------ now 15:20 --------------------   |                      |
| 18:00  [ ] Gym   [Gym Buddies]          |                      |
| [+ Add task]                            |                      |
+-----------------------------------------+----------------------+
```

## Functional requirements

| ID | Requirement |
| --- | --- |
| FR-D1 | Header: greeting, level + XP bar, fire icon with app streak (after the level), fortune button |
| FR-D2 | Today timeline: all-day tasks on top; timed tasks, group tasks (with group label) and timed habits by hour; a "now" line; mark done inline; add task button |
| FR-D3 | Overdue tasks bar, expands on click |
| FR-D4 | Today's habits: check in inline, count habits via [+] |
| FR-D5 | Fortune button: see [extras.md](extras.md) |

## Business rules

- App daily check-in gives +2 XP automatically on the first app open of the day, shown with a popup.
- App streak (fire): consecutive days opening the app. Missing one day resets it to 0 and the fire turns grey.
- Not on the Dashboard (they have their own pages): goals, groups, calorie, today's XP number, progress %.
