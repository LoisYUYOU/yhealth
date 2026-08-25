# YHealth Sync SOP

This file is the durable source of truth for updating YHealth from ChatGPT conversations.

## Trigger
Whenever the user says things like “记到 YHealth / 写入 / 记录一下”, or sends diet, sleep, activity, exercise, weight/body-fat, menstrual, bowel-movement, or other health data intended for YHealth, follow this SOP.

## Required workflow
1. Resolve the exact calendar date first. Convert “今天 / 昨天 / 前天” to an explicit YYYY-MM-DD based on the conversation timestamp and user wording. If the user explicitly corrects the date, the correction wins.
2. Read the existing `data/YYYY-MM-DD.json` before writing when it already exists. Merge new information; never overwrite unrelated prior records.
3. Normalize user input into the current YHealth schema. User wording and screenshots are the source; the user must not be forced to provide field names.
4. Diet: preserve each meal/snack separately with time, name, quantity/weight when known, calorie estimate/range, and estimate rationale. Later corrections replace earlier mistaken values.
5. Sleep: record duration, efficiency, stages, wakeups/awake time, heart rate, respiration, and other visible metrics when available. Unknown values must use frontend-safe null/placeholder handling; never create `undefined`-producing fields.
6. Daily activity: record dynamic kcal, steps, active minutes, stand/break hours and source time when available.
7. Exercise: use frontend-compatible keys. Canonical keys are `name`, `time`, `duration`, `active_kcal`, `total_kcal`, `avg_hr_bpm`, `max_hr_bpm`; additional training effect/load/recovery fields may be preserved. If the frontend still expects legacy keys, either update the frontend or write compatibility aliases. Never leave exercise name or HR as `undefined`.
8. Body: preserve bowel movements, weight, body fat and other body composition separately. Do not delete existing composition when adding bowel records or vice versa.
9. Ensure `data/profile.json` includes the date in `available_dates`. A valid daily JSON file that is absent from `available_dates` is considered an incomplete sync.
10. Validate frontend compatibility after each write. Check required nested objects and rendering assumptions in `index.html`; missing optional data must not crash the whole date page.
11. Re-read the written daily JSON after commit and verify at minimum: date, intake, diet, sleep, activity, exercise, body, and coach notes. Confirm no accidental loss, duplicate calorie counting, stale typo, `undefined`, or invalid null access.
12. A sync is only complete when both the GitHub data and the webpage are expected to load for that date. “File written successfully” alone is not completion.

## Precedence rules
- Explicit user correction > later user clarification > screenshot reading > earlier estimate.
- Screenshot data > guesswork.
- Unknown data stays unknown; do not fabricate precision.
- Daily dynamic calories already include exercise unless the source explicitly says otherwise. Exercise calories are detail-only and must not be added again to the daily total.

## Final confirmation format
Only after validation, tell the user that the date has been synced and checked. Mention any still-missing source data explicitly.

## Maintenance rule
Before any future YHealth write, read this file first if there is any uncertainty about the synchronization process or schema.