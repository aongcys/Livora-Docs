# Extras: Daily Fortune and Calorie

## Daily Fortune (tarot)

Entertainment only.

| ID | Requirement |
| --- | --- |
| FR-F1 | Press the fortune button on the Dashboard → shake animation → one card in a popup |
| FR-F2 | Popup shows the card image, card name and overall meaning (Thai / English per UI language) |
| FR-F3 | Popup footer: "เป็นเพียงการสุ่มเท่านั้น โปรดใช้วิจารณญาณ" / "This is only random. Please use your own judgement." |

Business rules:

- Deck: the 22 Major Arcana tarot cards, upright only. Card images are made in-house.
- One card per day; pressing again the same day shows the same card. The day resets at midnight in the user's timezone.
- The same card may repeat on other days.
- No XP. No history; only today's card is visible.
- Card content is seeded into the system (no admin edit page in V1).

Data needed: `fortune_cards` (name, image, meaning TH/EN) and a daily result (user, date, card), one row per user per day.

## Calorie

Only weight updates and the calories a user can eat per day.

| ID | Requirement |
| --- | --- |
| FR-K1 | First-time input: birth date, height, weight, activity level (5 levels); gender comes from the profile. Saved, not asked again |
| FR-K2 | Show calories per day (TDEE) as the main number and BMR as secondary |
| FR-K3 | Update weight at any time; recalculated instantly |
| FR-K4 | Weight history chart |
| FR-K5 | Edit height / activity level |

Business rules:

- BMR uses Mifflin-St Jeor; TDEE = BMR × activity factor.
- Age is computed from the birth date.
- No lose / gain goal, no food logging, no UNDER / OVER status, no XP.
- One weight value per day; entering again the same day replaces it.

Data needed: body profile (user, birth date, height, activity level) and weight history (user, date, weight). BMR / TDEE are computed at display time.

## Removed from scope

- Focus timer (removed 2026-11-13)
