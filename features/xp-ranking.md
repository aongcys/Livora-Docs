# 7. XP / Score

XP is the gamification system.

Recommended name in the product can be either:

- XP
- Score

Internally, prefer a clear term such as `xp`.

### Example XP rules

```text
Complete task        +10 XP
Complete habit       +15 XP
Daily check-in        +5 XP
Complete milestone   +25 XP
```

Exact values are configurable and should not be scattered throughout the codebase.

Use a centralized XP rule configuration/service.

### XP requirements

XP events should be recorded so the system can explain where XP came from.

Example:

```text
Task completed
+10 XP
```

This also helps prevent duplicate XP.

### Anti-abuse requirement

Completing the same task multiple times must not repeatedly award XP.

XP awarding should be idempotent.

---

# 8. Level

Level is derived from XP.

Example:

```text
Level 1
0 - 99 XP

Level 2
100 - 249 XP

Level 3
250 - 499 XP
```

The exact formula can be changed later.

Do not permanently duplicate derived values unless there is a performance reason.

---

# 9. Ranking

V1 uses a global ranking.

Do NOT implement groups or friends in V1 unless explicitly requested.

### V1 ranking

- Global leaderboard
- Weekly ranking
- Current user's rank
- XP earned during the ranking period

Example:

```text
Rank    User        XP

1       Alex       1240
2       Mark       1180
3       John       1120
12      Current     980
```

### Ranking period

V1 should primarily use weekly ranking.

The ranking period should be explicit, for example:

```text
week_start
week_end
```

Do not rank users using lifetime XP for the main competitive leaderboard because older accounts would have a permanent advantage.

### Future

V2 may add:

- Friends
- Groups
- Group leaderboard
- Challenges
- Private competitions
