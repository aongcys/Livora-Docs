# 23. Loading / Empty / Error States

Every major feature must consider:

### Loading

```text
Loading today's tasks...
```

### Empty

```text
No tasks today.
Create your first task.
```

### Error

```text
Something went wrong.
Try again.
```

### Success

```text
Task completed!
+10 XP
```

Do not design only the successful state.

---

# 24. UX Guidelines

The product should feel:

- Clean
- Friendly
- Lightweight
- Motivating
- Fast
- Easy to understand

Avoid making every screen look like a dashboard.

Use focused layouts for focused tasks.

Examples:

- Planner → information-dense
- Habit → visual/streak-oriented
- Ranking → competitive/table-oriented
- Dashboard → overview
- Focus → minimal/distraction-free

---

# 25. Gamification UX

Gamification should feel rewarding but not intrusive.

Good:

```text
Task completed ✓
+10 XP
```

Avoid:

- Excessive animations
- Constant popups
- Too many badges
- Forced social interaction

Users should still be able to use the product as a productivity tool.

---

# 28. Definition of Done

A feature is not complete merely because the API works.

Each feature should include:

- Database model
- API
- Validation
- Authorization
- Frontend UI
- Loading state
- Empty state
- Error state
- Success state
- Responsive behavior
- Basic edge-case handling
- Tests where appropriate
