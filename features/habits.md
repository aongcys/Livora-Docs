# 6. Habit

A Habit represents something the user wants to repeat consistently.

Examples:

- Exercise
- Read a book
- Drink water
- Study
- Sleep before a specific time

A Habit is different from a Task.

Task:

```text
Run on September 20
```

Habit:

```text
Run
Every Tuesday and Thursday
```

### Core features

- Create habit
- Edit habit
- Delete habit
- Set schedule
- Daily check-in
- Undo check-in where appropriate
- Habit history
- Streak
- Current streak
- Best streak
- Completion rate
- Calendar history
- Optional reminder

### Habit schedule

A habit may have:

```text
DAILY
WEEKLY
CUSTOM
```

The system should create or calculate occurrences without duplicating unnecessary permanent rows.

### Streak

The system should track:

- Current streak
- Best streak
- Last completed date

A streak should be based on scheduled occurrences, not simply consecutive calendar days.

Example:

A habit scheduled only Monday and Thursday should not lose its streak because Tuesday was not scheduled.
