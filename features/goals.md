# 10. Goals

Goals represent larger objectives.

Examples:

```text
Run 100 KM
Read 10 books
Finish project
Study 20 hours
```

### Core features

- Create goal
- Edit goal
- Delete goal
- Set start date
- Set deadline
- Set target
- Track current progress
- View progress percentage
- Goal status

### Goal status

```text
NOT_STARTED
IN_PROGRESS
COMPLETED
FAILED
CANCELLED
```

### Goal progress

A goal should support a measurable target where possible.

Example:

```text
Goal:
Run 100 KM

Current:
72 KM

Target:
100 KM

Progress:
72%
```

Recommended fields:

```text
target_value
current_value
unit
```

Do not force every future goal to use the same unit.
