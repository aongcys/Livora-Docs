# 12. goals

```text
goals
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id

name                text
description         text nullable

start_date          date
deadline            date nullable

target_value        numeric nullable
current_value       numeric nullable
unit                text nullable

status              goal_status

created_at          timestamptz
updated_at          timestamptz
```

Enum:

```text
goal_status
NOT_STARTED
IN_PROGRESS
COMPLETED
FAILED
CANCELLED
```

Progress can be calculated:

```text
progress = current_value / target_value
```

The backend should clamp display progress to 0–100%.
