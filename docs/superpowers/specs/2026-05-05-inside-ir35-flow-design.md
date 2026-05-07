# Inside IR35 Question Flow & Calculations — Design Spec
Date: 2026-05-05

## Overview
Add the full Inside IR35 question flow and output screen to the existing single-file PWA (`index.html`). The existing UI shell, design system, and navigation framework are not modified beyond extending them.

## New Screens

| Screen ID | Question | Input Type | Condition |
|-----------|----------|------------|-----------|
| screen-2 | What is your day rate? | Number input (£) | Always (existing) |
| screen-3 | How many days do you work per week? | Tappable cards 1–5 | Inside IR35 |
| screen-4 | How many days holiday do you take per year? | Number input | Inside IR35 |
| screen-5 | What is your umbrella company margin fee per week? | Number input (£), default £20 | Inside IR35 |
| screen-6 | Do you pay into a pension? | Yes/No cards | Inside IR35 |
| screen-7 | What percentage of gross do you contribute? | Number input (%) | Inside IR35 + pension=Yes |
| screen-8 | Are you employed elsewhere? | Yes/No cards + conditional warning banner | Inside IR35 |
| screen-9 | Output — Your IR35 Estimate | Results display | Inside IR35 |

Screen-1 (IR35 choice) remains as-is. Outside IR35 routes are for future stages.

## Navigation Architecture

Replace the current linear `CFG` array with a `getNext()` / `getPrev()` approach using a history stack:

- `history` array tracks the sequence of visited screen indices
- `goBack()` pops from history
- `onContinue()` calls `getNextScreen(cur, ans)` which returns the next screen index
- `getNextScreen` implements branching:
  - screen-1 (IR35 choice) → screen-2 always
  - screen-6 (pension) → screen-7 if pension=yes, else screen-8
  - screen-7 (pension %) → screen-8
  - screen-8 (employed elsewhere) → screen-9

## Tax Calculations (2026/27 rates)

```
annual_gross        = day_rate × working_days
working_days        = (52 × days_per_week) − holiday_days

umbrella_fee_annual = umbrella_fee_weekly × 52
employer_ni         = max(0, (annual_gross − 5000) × 0.15)
apprenticeship_levy = annual_gross × 0.005
employee_gross      = annual_gross − umbrella_fee_annual − employer_ni − apprenticeship_levy

personal_allowance  = 12570 (tapers above £100k: PA − max(0, (employee_gross − 100000)/2), floor 0)
taxable_income      = max(0, employee_gross − personal_allowance)
income_tax          = band(0→37700: 20%) + band(37700→112570: 40%) + band(112570+: 45%)

employee_ni:
  on £12,570–£50,270: 8%
  above £50,270:       2%

pension_contribution = (pension=yes) ? employee_gross × (pension_pct/100) : 0

net_take_home = employee_gross − income_tax − employee_ni − pension_contribution
```

## Output Screen Content

- Gross contract value (annual_gross)
- Umbrella fee annual (red, deduction)
- Employer NI (red, highlighted — "money that never reaches you")
- Apprenticeship Levy (red)
- Employee NI (red)
- Income Tax (red)
- Pension contribution (red, shown only if pension=yes)
- Net annual take-home (hero, green, DM Serif Display)
- Net monthly take-home (annual / 12)
- Effective tax rate ((total deductions / annual_gross) × 100)
- Disclaimer text
- Numbers count up from 0 over 800ms on reveal
- Cards stagger in with 100ms delay between each

## Employed Elsewhere Warning
If user selects Yes on screen-8, a warning banner appears below the cards on the same screen (does not block continuing). Banner text: "Your Personal Allowance and NI thresholds may already be used by your employment. The figures shown may not reflect your actual liability — speak to an accountant."

## Constraints
- No external libraries
- Single file — all HTML/CSS/JS inline
- Design tokens unchanged (colours, fonts, border radii, safe area insets)
- All touch targets ≥ 48px
- Number inputs trigger numeric keyboard on iPhone (`inputmode="numeric"`)
