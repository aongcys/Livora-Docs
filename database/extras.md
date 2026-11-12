# 17. Daily Fortune

```text
daily_fortunes
------------------------------
id                  uuid PK
user_id             uuid FK -> users.id
fortune_date        date

overall_rating      integer
work_rating         integer nullable
money_rating        integer nullable
love_rating         integer nullable
health_rating       integer nullable

message             text
lucky_number        integer nullable
lucky_color         text nullable

created_at          timestamptz
```

Recommended unique constraint:

```text
(user_id, fortune_date)
```

This ensures a user receives one stable fortune per day.

---

# 18. Calorie Profile

User's calculation inputs.

```text
calorie_profiles
------------------------------
id                  uuid PK
user_id             uuid UNIQUE

age                 integer
sex                 calorie_sex
height_cm           numeric
weight_kg           numeric
activity_level      calorie_activity_level

created_at          timestamptz
updated_at          timestamptz
```

Enums:

```text
calorie_sex
MALE
FEMALE
OTHER
```

```text
calorie_activity_level
SEDENTARY
LIGHT
MODERATE
ACTIVE
VERY_ACTIVE
```

The exact calorie formula belongs in backend business logic.

Pending (from FRS): replace `age` with birth date, and move weight into a weight history table (one value per day). See [features/extras.md](../features/extras.md).
