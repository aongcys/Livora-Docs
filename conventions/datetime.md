# 21. Date and Time Rules

The application is daily/time-based, so date handling must be explicit.

Do not blindly use server local time.

Store timestamps consistently and convert to the user's timezone for display.

The user's timezone should be part of their profile/settings.

Important cases:

- Recurring habits
- Daily fortune
- Daily group ranking (resets at midnight Thai time)
- Reminder scheduling
- Calendar display
- Day boundaries
- Daylight-saving changes if international users are supported later

For V1, the application may target a single primary timezone but should avoid hard-coding timezone logic into business rules.
