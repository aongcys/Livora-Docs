# 12. Daily Fortune

Daily Fortune is an entertainment feature.

It can provide randomized content such as:

- Overall luck
- Work
- Money
- Health
- Love
- Lucky number
- Lucky color
- Daily message

The system should treat this as entertainment/random content, not factual prediction.

### Important

The user should receive one daily result rather than a completely new result every refresh.

Example:

```text
User opens on Sep 20
→ generate/store today's fortune

User refreshes
→ same fortune

Sep 21
→ new fortune
```

This prevents refresh abuse and makes the feature feel like a daily event.

---

# 13. Calorie

V1 should keep calorie functionality lightweight.

### Initial scope

Calculate:

- BMR
- TDEE
- Daily calorie target

Input:

- Age
- Sex
- Height
- Weight
- Activity level
- Goal

Example:

```text
BMR
TDEE
Target Calories
```

### Calorie status

The daily calorie page may show:

```text
UNDER
ON_TARGET
OVER
```

Example:

```text
Target: 2000 kcal
Consumed: 2150 kcal

Status: OVER
```

Do not build a large food database in V1.

---

# 14. Focus Time

Focus Time is a lightweight productivity timer.

Example:

```text
Task: Study Machine Learning

25:00

[ Start ]
```

Possible V1 timer:

- Start
- Pause
- Resume
- Complete
- Reset

After a completed focus session, it may award XP.

The focus session should optionally link to a Task.
