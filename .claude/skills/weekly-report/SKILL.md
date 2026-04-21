---
name: weekly-report
description: Interactively fill the ARCTICOM weekly status report. Use when the user says "weekly report", "fill status report", "/weekly-report", or asks to generate this week's timesheet/status HTML. Default target week is the current week (Mon–Fri).
---

# Weekly Report — Interactive Filler

Generate a filled `weekly-status-report.html` from the template at `template/weekly-status-report.html` based on an interactive back-and-forth with the user.

## Paths

- Template: `/Users/yuantingluo/census-pm/template/weekly-status-report.html`
- Output dir: `/Users/yuantingluo/census-pm/reports/`
- Output filename: `weekly-status-<YYYY-MM-DD>.html` where `<YYYY-MM-DD>` is the **Friday** week-ending date.

## Flow

Follow these steps in order. Do not skip steps. Ask the user one focused question per turn — do not dump all questions at once.

### Step 1 — Compute the target week

Default target = **current week**. Determine the Friday of this week:

```bash
python3 -c "
import datetime
today = datetime.date.today()
# Monday=0 ... Friday=4 ... Sunday=6
friday = today + datetime.timedelta(days=(4 - today.weekday()) % 7 if today.weekday() <= 4 else -(today.weekday() - 4))
monday = friday - datetime.timedelta(days=4)
days = [(monday + datetime.timedelta(days=i)) for i in range(5)]
print('friday=' + friday.isoformat())
print('week_ending=' + friday.strftime('%m/%d/%Y'))
for i, d in enumerate(days):
    print(f'day{i+1}={d.strftime(\"%A\")}|{d.strftime(\"%m/%d/%Y\")}')
"
```

Show the computed week ending to the user and ask: *"Filling report for week ending <mm/dd/yyyy>. Proceed, or pick a different Friday?"*

If they want a different week, accept an ISO date or "last week"/"next week" and recompute.

### Step 2 — Collect high-level week summary

Ask: *"Give me a high-level description of what you worked on this week. I'll break it down into Monday–Friday for you."*

Wait for their response. This is usually a paragraph or bullet list.

### Step 3 — Break down into days

Based on their summary, propose **one-line activities for each of Monday–Friday**. Distribute work realistically — don't just copy the same line five times. Use judgment:

- If they mention specific things (meetings, reviews, deployments), anchor those to likely days.
- Spread longer-running work (feature dev, bug fixing) across multiple days.
- Keep each line to 1–2 short sentences — this is a status report, not a journal.

Present the proposed breakdown as a table and ask: *"Does this breakdown look right? Any edits?"*

Iterate until confirmed.

### Step 4 — Collect hours

Ask: *"How many hours did you work each day? You can give me a per-day breakdown (e.g. `8,8,7,8,8`) or a total and I'll distribute evenly."*

Capture `hours_1`..`hours_5`.

### Step 5 — Upcoming activities

Ask: *"What's on deck for the next two weeks? (free-form — meetings, testing, deployments, etc.)"*

Capture `upcoming_activities`.

### Step 6 — Hours reconciliation

Compute `sum(hours_1..hours_5)`. Ask the user for the **Actual Hours** figure that goes in the top table (this is typically the contract-hours total logged that week).

If `actual_hours != sum_of_daily_hours`, **flag it**:

> ⚠️ Mismatch: daily hours sum to X but Actual Hours is Y. Which is correct?

Do not write the file until this is resolved (either the user adjusts a day, changes the actual, or explicitly acknowledges the mismatch is intentional).

### Step 7 — Write the filled HTML

Read the template, substitute these placeholders with the collected values:

- `{{week_ending}}` → Friday date as `mm/dd/yyyy`
- `{{day_1}}`..`{{day_5}}` → weekday names (Monday..Friday)
- `{{date_1}}`..`{{date_5}}` → per-day dates as `mm/dd/yyyy`
- `{{activities_1}}`..`{{activities_5}}` → the activity lines
- `{{hours_1}}`..`{{hours_5}}` → per-day hours
- `{{actual_hours}}` → the reconciled actual hours
- `{{upcoming_activities}}` → free-form text

Write to `/Users/yuantingluo/census-pm/reports/weekly-status-<YYYY-MM-DD>.html`.

**Important:** the template references `../logo.png`. Since the output is in `reports/` (sibling of `template/` and `logo.png`), the reference still resolves correctly.

### Step 8 — Auto-open

Open the file for review:

```bash
open "/Users/yuantingluo/census-pm/reports/weekly-status-<YYYY-MM-DD>.html"
```

Tell the user: *"Opened <path>. Review in the browser, then Cmd+P → Save as PDF when ready."*

## Notes

- Never modify the template itself.
- If `reports/` doesn't exist, create it.
- If a file for the same Friday already exists, ask before overwriting.
- Keep your questions short. The user wants to fill this fast, not answer a survey.
