---
name: weekly-report
description: Interactively fill the Team Katmai weekly status report. Use when the user says "weekly report", "fill status report", "/weekly-report", or asks to generate this week's status HTML. Default target week is the current week (ending Friday).
---

# Weekly Report — Interactive Filler

Generate a filled `weekly-status-report.html` from the template at `template/weekly-status-report.html` based on an interactive back-and-forth with the user.

The report is the **Team Katmai Weekly Status Report** — four short narrative answers, due each Friday COB.

## Paths

- Template: `/Users/yuantingluo/census-pm/template/weekly-status-report.html`
- Output dir: `/Users/yuantingluo/census-pm/reports/`
- Output filename: `weekly-status-<YYYY-MM-DD>.html` where `<YYYY-MM-DD>` is the **Friday** week-ending date.

## Fixed values

Unless the user says otherwise:

- `{{name}}` → `Yuanting Luo`
- `{{position}}` → `Application Developer`

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
print('friday=' + friday.isoformat())
print('week_ending=' + friday.strftime('%m/%d/%Y'))
print('week_start=' + monday.strftime('%m/%d/%Y'))
"
```

Show the computed week ending to the user and ask: *"Filling report for week ending <mm/dd/yyyy>. Proceed, or pick a different Friday?"*

If they want a different week, accept an ISO date or "last week"/"next week" and recompute.

Also check `reports/` for the **previous** week's file — Step 3 ("what changed") is relative to it, so read it if present.

### Step 2 — Collect the week in free form

Ask: *"What did you work on this week, and where does that leave things? I'll shape it into the four answers."*

Wait for their response. A sentence or a few bullets is enough — this report is short.

### Step 3 — Draft the four answers

From their summary, draft all four answers and present them together for review. Keep each one tight — this format rewards brevity.

1. **Where we are.** One sentence against the plan. Anchor it to a milestone, sprint, or release if the user has mentioned one; otherwise state the current state of the main workstream.
2. **What changed.** The one thing that's different since last week. Compare against the previous week's report if one exists in `reports/`. **If nothing changed, write exactly "Nothing changed."** — do not pad it.
3. **What I need.** Blockers, decisions, access, or reviews needed from others. **If nothing, write "Nothing."** — do not invent asks.
4. **Upcoming Planned Absences.** Known PTO or absences of **more than 1 day**. If none, write "None." Single days off (including holidays) do not belong here.

Present as a short list and ask: *"Look right? Any edits?"*

Iterate until confirmed. Do not move on with placeholder text in any of the four.

### Step 4 — Write the filled HTML

Read the template, substitute these placeholders:

- `{{name}}` → name (default `Yuanting Luo`)
- `{{position}}` → position (default `Application Developer`)
- `{{week_ending}}` → Friday date as `mm/dd/yyyy`
- `{{where_we_are}}` → answer 1
- `{{what_changed}}` → answer 2
- `{{what_i_need}}` → answer 3
- `{{planned_absences}}` → answer 4

Escape any `&`, `<`, `>` in the answers. Assert no `{{` remains before writing.

Write to `/Users/yuantingluo/census-pm/reports/weekly-status-<YYYY-MM-DD>.html`.

**Important:** the template references `../assets/katmai-logo.png`. Since the output lands in `reports/` (sibling of `assets/`), the reference still resolves correctly.

### Step 5 — Auto-open

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
- Reports before 10/2026 use the older ARCTICOM format (daily activity table + hours). Those stay as-is; do not reformat them.
