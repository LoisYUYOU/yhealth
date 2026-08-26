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
7. Exercise: use frontend-compatible keys. Canonical keys are `name`, `time`, `duration`, `active_kcal`, `total_kcal`, `avg_hr_bpm`, `max_hr_bpm`; additional training effect/load/recovery fields may be preserved. The current frontend accepts canonical keys and legacy aliases, but canonical keys should be written first. Never leave exercise name or HR as `undefined`.
8. Body: preserve bowel movements and body composition independently. Canonical body-composition path is `body.composition`. Recommended fields are `time`, `weight_kg`, `bmi`, `body_fat_percent`, `muscle_mass`, `source`, and `notes`. Unknown metrics must remain `null`; never invent values. Adding or updating weight/body composition must not remove `body.bowel_movements`, and adding a bowel record must not remove `body.composition`.
9. Body specialty rendering: whenever a weight or body-composition record is added, verify that the `身体` tab shows the `体重专项` card for that date. The card should show weight prominently, measurement time, BMI, body-fat percentage and muscle mass when available, plus a weight trend generated from all dates that contain `body.composition.weight_kg`. The overview page should also show the current day's weight when available.
10. Ensure `data/profile.json` includes the date in `available_dates`. A valid daily JSON file that is absent from `available_dates` is considered an incomplete sync.
11. Validate frontend compatibility after each write. Check required nested objects and rendering assumptions in `index.html`; missing optional data must not crash the whole date page. Specifically verify diet, sleep, activity, exercise, body composition, bowel movements and coach notes render without `undefined`, `null` text leakage or invalid property access.
12. Re-read the written daily JSON after commit and verify at minimum: date, intake, diet, sleep, activity, exercise, body, and coach notes. Confirm no accidental loss, duplicate calorie counting, stale typo, `undefined`, or invalid null access.
13. If the data schema adds a new project/specialty that the frontend does not render yet (for example weight/body composition), update `index.html` as part of the same task. A data-only write is incomplete when the user expects to see the data on the webpage.
14. After changing `index.html`, verify the page still supports all existing dates and both canonical and historical field names where needed. New frontend logic must be null-safe so partial daily records can load.
15. A sync is only complete when both the GitHub data and the webpage are expected to load for that date. “File written successfully” alone is not completion.

## Body / weight project update checklist
When the user sends a scale screenshot or body-composition data:
1. Read this SOP and the day's existing JSON.
2. Extract only visible/explicit measurements. Typical fields: measurement time, weight, BMI, body-fat percentage, muscle mass and comparison-to-prior-day note.
3. Merge those values into `body.composition`; retain existing bowel records and other body data.
4. If the date is new, create a complete frontend-safe daily JSON and add the date to `data/profile.json.available_dates`.
5. Re-read the daily JSON and verify `body.composition.weight_kg` and any provided metrics are present.
6. Check `index.html` still renders: `身体 → 体重专项`, measurement time, available composition metrics, and the cross-day `体重趋势` chart. The overview should show the day's weight when available.
7. If any of those views are absent, update the frontend before declaring synchronization complete.

## Precedence rules
- Explicit user correction > later user clarification > screenshot reading > earlier estimate.
- Screenshot data > guesswork.
- Unknown data stays unknown; do not fabricate precision.
- Daily dynamic calories already include exercise unless the source explicitly says otherwise. Exercise calories are detail-only and must not be added again to the daily total.

## Final confirmation format
Only after validation, tell the user that the date has been synced and checked. Mention any still-missing source data explicitly.

## Maintenance rule
Before any future YHealth write, read this file first if there is any uncertainty about the synchronization process or schema. If the user explicitly asks to follow the SOP, reading this file is mandatory before writing.