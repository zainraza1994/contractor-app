# Contractor Calculator — Project Context

## What this project is
A UK contractor take-home pay calculator built as a mobile-first PWA (Progressive Web App).
It is a single HTML file with vanilla JavaScript and CSS — no frameworks, no libraries, no backend.
It is hosted on GitHub Pages and tested on iPhone Safari.

## Who built it and how
The owner is a non-developer building this with Claude Code assistance.
All code is written by Claude Code based on plain English instructions.
Do not assume prior coding knowledge when explaining anything — always explain clearly.

## What the app does
Steps a UK contractor through a series of questions and calculates their net take-home pay.
Supports three scenarios:
- Inside IR35 (via umbrella company)
- Outside IR35 — Sole Trader
- Outside IR35 — Limited Company (single or multiple directors/shareholders)

For the Limited Company route, supports up to 3 directors with individual shareholding
percentages, salaries, and personal tax calculations. Outputs both per-director breakdowns
and a combined total take-home across all directors.

## Tax rates — always use 2026/27 UK rates
- Personal Allowance: £12,570 (frozen until 2031)
- Income Tax: 20% on £12,571–£50,270 / 40% on £50,271–£125,140 / 45% above £125,140
- Personal Allowance tapers to zero above £100,000
- Employee NI: 8% on £12,570–£50,270 / 2% above
- Employer NI: 15% above £5,000/year (secondary threshold £96/week)
- Apprenticeship Levy: 0.5% of gross (inside IR35 only)
- Corporation Tax: 19% under £50k / marginal relief £50k–£250k (26.5% effective) / 25% above £250k
- Dividend Allowance: £500
- Dividend Tax: 10.75% basic / 35.75% higher / 39.35% additional
- Class 4 NI (sole trader): 6% on £12,570–£50,270 / 2% above
- Class 2 NI: voluntary only, do not include by default

## Student loan repayment rates — 2026/27
Source: HMRC SL3 Student and Postgraduate Loan deduction tables, 2026 to 2027.
These thresholds are uprated most Aprils — re-verify them at the start of each tax year.
- Plan 1: 9% above £26,900
- Plan 2: 9% above £29,385
- Plan 4: 9% above £33,795
- Plan 5: 9% above £25,000
- Postgraduate Loan: 6% above £21,000
- Income base: grossSalary for Inside IR35; salary + dividendsReceived per director for Ltd Co
- Do not include student loan in the sense check or pot calculation — it is deducted after net salary is calculated

## Design rules — never change these
- Background: deep navy #0D1B2A
- Accent / CTAs / positive numbers: emerald green #00C48C
- Deductions / warnings: soft red #FF6B6B
- Cards and text: white
- Fonts: DM Sans (body/UI) and DM Serif Display (headline numbers) from Google Fonts CDN
- One question per screen — Typeform style, never a scrolling form
- Page transitions: 320ms cubic-bezier(0.4, 0, 0.2, 1), slides left/right directionally
- Progress bar: 4px, top of screen, fills in green
- Continue button: fixed to bottom, pill shape, full width, disabled until field valid
- Back button: top left, lightweight chevron + text, no background
- Cards: 16px border radius, subtle shadow only
- Pill buttons: 100px border radius
- Input cards: 12px border radius
- All touch targets: minimum 48px
- Safe area insets applied throughout (env(safe-area-inset-*))
- No scrollbars — every screen fits within the viewport
- Numbers on output screen count up from 0 over 800ms on reveal
- Output cards stagger in with 100ms delay between each

## File structure
- Single file: index.html
- All CSS and JavaScript inline in the same file
- Service worker and PWA manifest included inline
- No external dependencies except Google Fonts CDN

## Deployment
- GitHub Pages — every push to main goes live automatically
- Live URL: [add your GitHub Pages URL here once set up]

## Key rules when making changes
- Never change the tax rates without being explicitly asked
- Tax rates live only in the `RATES` object and `STUDENT_LOAN_PLANS` — never hardcode a rate inside a calculation function
- Never introduce external libraries or frameworks
- Never break the single-file structure
- Never change the colour palette or fonts
- Always test that number inputs trigger numeric keyboard on iPhone
- Always maintain safe area insets for iPhone notch
- Always keep the one-question-per-screen layout
- When adding new screens, follow the existing transition pattern exactly
- When fixing a bug, explain what caused it in plain English after fixing it

## Current build status
Built in 6 stages:
- [x] Stage 1: UI shell and design system
- [x] Stage 2: Inside IR35 route and calculations
- [ ] Stage 3: Sole Trader route and calculations
- [x] Stage 4: Limited Company, single director
- [x] Stage 5: Limited Company, multiple directors (up to 3)
- [x] Stage 6: Comparison screen, PWA install, final polish

## Navigation architecture (important for future changes)
The JavaScript uses a **history-stack navigation** — not a linear index. Key pieces:

- `SCREENS` — object map (keys 0–31) with per-screen config: progress %, step label, button text, and an `ok()` function that enables/disables Continue. Progress % and step label for screens 20–28 are functions (not fixed strings) that compute dynamically based on `dirLoop.current`.
- `history` — array stack; `onContinue()` pushes current index before advancing; `goBack()` pops and returns
- `getNextScreen(idx)` — handles all routing branches:
  - Screen 1 → screen 10 if `ans.ir35 === 'outside'`, else screen 2 (Inside IR35)
  - Screen 6 → screen 7 if `ans.pension === 'yes'`, else screen 8
  - Screen 8 → screen 26 (student loan); screen 26 → screen 9
  - Screen 13 → screen 30 (VAT status — always, for all director counts)
  - Screen 30 → screen 31 if `ans.vatStatus === 'flatrate'`; else → 14 (1 director) or calls `initDirLoop()` and → 20
  - Screen 31 → screen 14 (1 director) or calls `initDirLoop()` and → 20
  - Screen 16 → screen 17 for single director (pension yes/no) — this was previously broken and is now fixed
  - Screen 17 → screen 18 if ltdPension='yes', else screen 27; screen 18 → screen 27; screen 27 → screen 19
  - Screen 24 → screen 25 if current director's pension is 'yes'; otherwise screen 28
  - Screen 25 → screen 28; screen 28 → `saveCurrentDirector()` then `nextDirectorOrExit()` (returns 20 if more directors remain, 16 when all done)
- `ans` object — stores all answers: `{ ir35, daysPerWeek, pension, employedElsewhere, ltdDaysPerWeek, numDirectors, ltdEmployedElsewhere, ltdPension, vatStatus, directors: [] }`
- `dirLoop = { current: 0 }` — tracks which director (0-indexed) the loop is currently collecting data for
- `goBack()` decrements `dirLoop.current` at two cross-director boundary crossings:
  1. Navigating back to screen 20 from screen 24, 25, or 28 (between directors mid-loop)
  2. Navigating back from screen 16 to screen 28 (re-entering the loop after all directors completed — `dirLoop.current` would otherwise be out-of-bounds at `numDirectors`)
  - In both cases, `populateLoopScreens()` is called immediately after decrementing so the shared DOM inputs (name, shareholding, salary, pension amount) reflect the correct director's saved data before any further saves via `saveCurrentDirector()`
- `loopPct(fieldIdx)` now uses 7 fields (0=name, 1=shareholding, 2=salary, 3=employed, 4=pension, 5=pension-amount, 6=student-loan)
- Both output screens (9 and 19) use "Start Over" which calls `resetAll()` and returns to screen 0
- `goToComparison()` — called by the ghost pill button on screens 9 and 19; pushes current screen to history then goes to screen 29
- Screen 29 `onContinue` calls `resetAll()` and goes to screen 0 (same as screens 9 and 19)

**When adding the Sole Trader route:** screens 20–25 are taken by the director loop. Add Sole Trader screens from 26+. Update `getNextScreen(1)` to branch on `ans.ir35 === 'sole-trader'`, and add a new `ir35` card value for sole trader on screen 1.

## Screen map — Inside IR35 (screens 0–9)
| Index | ID | Content |
|---|---|---|
| 0 | screen-0 | Welcome |
| 1 | screen-1 | IR35 choice (inside / outside) |
| 2 | screen-2 | Day rate (£ input) |
| 3 | screen-3 | Days per week (tappable cards 1–5) |
| 4 | screen-4 | Days holiday per year (number input) |
| 5 | screen-5 | Umbrella fee per week (£ input, default £20) |
| 6 | screen-6 | Pension? Yes/No cards |
| 7 | screen-7 | Pension % (conditional — only if pension=yes) |
| 8  | screen-8  | Employed elsewhere? Yes/No + warning banner if yes |
| 26 | screen-26 | Student loan? (6-card 2-col grid: No/Plan1/Plan2/Plan4/Plan5/Postgrad) |
| 9  | screen-9  | IR35 output — full calculated breakdown |

## Screen map — Outside IR35 Ltd Co (screens 10–25)

### Shared entry (all director counts)
| Index | ID | Content |
|---|---|---|
| 10 | screen-10 | Day rate (£ input, id `ltd-day-rate`) |
| 11 | screen-11 | Days per week (tappable cards, data-group `ltdDaysPerWeek`) |
| 12 | screen-12 | Days holiday per year (number input, id `ltd-holiday-days`) |
| 13 | screen-13 | Number of directors (cards 1–3, data-group `numDirectors`) |

### Single director only (screens 14–18 + 27, skipped for 2+ directors)
| Index | ID | Content |
|---|---|---|
| 14 | screen-14 | Director salary (£ input, id `ltd-salary`, default £12,570) |
| 15 | screen-15 | Employed elsewhere? Yes/No + warning banner (data-group `ltdEmployedElsewhere`) |
| 17 | screen-17 | Company pension? Yes/No cards (data-group `ltdPension`) |
| 18 | screen-18 | Monthly pension amount (£ input, id `ltd-pension-amount`, conditional) |
| 27 | screen-27 | Student loan? (6-card 2-col grid) |

### Shared expenses (all director counts)
| Index | ID | Content |
|---|---|---|
| 16 | screen-16 | Monthly company expenses — single-column scrollable list of 9 category inputs (see below) |
| 19 | screen-19 | Ltd Co output — Company card + per-director cards + summary |

### Director detail loop (screens 20–28, 2+ directors only — repeated once per director)
| Index | ID | Content |
|---|---|---|
| 20 | screen-20 | Director name (text input, id `dir-name`) |
| 21 | screen-21 | Shareholding % (number input, id `dir-shareholding`; must total 100% across all directors). **Last director only:** input is auto-filled with `100 − allocatedSoFar()`, locked read-only, hint reads "Auto-calculated as the remaining share" in green, Continue enabled immediately. |
| 22 | screen-22 | Director salary (£ input, id `dir-salary-loop`, default £12,570) |
| 23 | screen-23 | Employed elsewhere? Yes/No cards (data-group `dirEmployedElsewhere`) |
| 24 | screen-24 | Company pension? Yes/No cards (data-group `dirPension`) |
| 25 | screen-25 | Monthly pension amount (£ input, id `dir-pension-amount-loop`, conditional) |
| 28 | screen-28 | Student loan? (6-card 2-col grid; data-group `dirStudentLoan`) |

The loop label `dir-loop-lbl-{20–25}` on each screen shows "Director N of M" and is updated by `populateLoopScreens()` when entering screen 20.

### VAT screens (screens 30–31, Ltd Co only — inserted between screen 13 and screen 14/20)
| Index | ID | Content |
|---|---|---|
| 30 | screen-30 | VAT Status — 3 tappable cards: "Not VAT registered" (`none`), "Standard VAT" (`standard`), "Flat Rate Scheme" (`flatrate`). Stored as `ans.vatStatus`. |
| 31 | screen-31 | VAT Flat Rate % — number input (`id="vat-flat-rate"`), shown only when `vatStatus === 'flatrate'`. |

**Navigation:** `getNextScreen(13)` → 30. `getNextScreen(30)` → 31 if flatrate, else 14 (1 director) or 20 (multi, calling `initDirLoop()`). `getNextScreen(31)` → 14 (1 director) or 20 (multi, calling `initDirLoop()`).

**FRS Calculation:** `frsSurplus = max(0, annualGross × (0.20 − 1.20 × flatRate))`. Added to `taxableProfit` before corporation tax. Stored in `ltdCalcResult.frsSurplus`. Shown as "VAT Flat Rate Surplus" (positive, emerald) in the Company card on screen 19, hidden when zero.

### Comparison output (screen 29)
| Index | ID | Content |
|---|---|---|
| 29 | screen-29 | Comparison — stacked IR35 vs Ltd Co cards, Start Over button |

**Screen 21 — last-director auto-fill:** `onEnter(21)` checks `dirLoop.current === +ans.numDirectors − 1`. If true, it sets `dir-shareholding` to `100 − allocatedSoFar()`, marks the input `readOnly = true`, dims it (`opacity: 0.6`, `pointerEvents: none`), updates the hint in emerald, and calls `syncUI()`. `syncShareholding()` skips its normal hint update when the input is `readOnly`.

## Screen 16 — Expense categories
Screen 16 is a single-column scrollable list of 9 input cards, all blank by default. The `EXP_CATS` array
(defined at the top of the IIFE) is the single source of truth for IDs and labels:

| Input ID | Label (output) | Card display |
|---|---|---|
| `exp-accounting` | Accounting fees | Accounting fees |
| `exp-travel` | Travel | Travel |
| `exp-insurance` | Insurance | Insurance |
| `exp-equipment` | Equipment | Equipment |
| `exp-software` | Software & subscriptions | Software & subs |
| `exp-phone` | Phone & communications | Phone & comms |
| `exp-food` | Food & entertainment | Food & entertain |
| `exp-wages` | Other wages | Other wages |
| `exp-misc` | Miscellaneous | Miscellaneous |

- A live "Total: £X/month" line (id `exp-total-lbl`, styled in `--emerald`) updates via `syncUI()`.
- `getExpTotal()` sums all 9 inputs; SCREENS[16].ok() requires total ≥ 1.
- `calculateLtdCo()` reads expenses via `EXP_CATS` → stores both the total and an `expBreakdown` array in its return value.
- **Do not add a default value to any expense input** — they are intentionally blank.
- **Do not replace with a single input** — the categorised list is the intended design.
- **Fixed total bar:** `#exp-total-lbl` lives *outside* all `.screen` divs (placed just before `#chrome-bot`) so it can use `position:fixed`. If it were inside a `.screen` (which has `will-change:transform`), fixed positioning would be relative to the screen element, not the viewport. `syncUI()` shows/hides it (`display:block`/`none`) based on whether the current screen is 16. `#screen-16` has extra `padding-bottom` (160px + safe area) to keep the last card clear of the fixed total bar.

## Ltd Co output layout (screen 19)
Card order (top to bottom):
- **Hero card** — Net Annual Take-Home (count-up animation; combined total for 2+ directors)
- **Net Monthly Take-Home card** (`#o-ltd-monthly-card`) — always visible (both 1 and 2+ directors); shows `totalNetTakeHome / 12`; first card in `.out-cards`, mirroring the Inside IR35 output layout
- **Company card** — Gross Revenue, VAT Flat Rate Surplus (hidden when zero; positive/emerald; id `o-ltd-vat-frs-row`), Expenses (with indented per-category breakdown beneath for non-zero categories, populated by `renderLtdOutput()` into `#o-ltd-expenses-breakdown`; breakdown text uses `rgba(13,27,42,.45)` — dark, not white), Total Salaries, Total Pensions (hidden if none), Employer NI (hidden if zero — always zero for 2+ directors at £12,570 salary due to employment allowance), Corporation Tax, Dividends Available
- **Director cards** — one per director, generated dynamically into `#director-cards-container` (which has `display:flex;flex-direction:column;gap:10px` to space cards correctly). Each shows: Gross Salary / Net Salary (two-column), Dividends Received, Dividend Tax, Director Net Take-Home. Header shows "Director" for 1 director; "Name · X%" for 2+.
- **Combined Total card** (`#o-combined-card`) — visible for 2+ directors only; shows Total Net Take-Home, Total Tax Paid, Combined Effective Rate
- **Effective Tax Rate** card — visible for 1 director only
- Disclaimer

### Ltd Co calculation notes
- **Employment Allowance:** £10,500 for 2+ directors; £0 for single director. Applied to Employer NI before deducting from taxable profit.
- **Employer NI per director:** `max(0, salary − £5,000) × 15%`. Summed across all directors, then employment allowance subtracted.
- **VAT Flat Rate Surplus:** `frsSurplus = max(0, annualGross × (0.20 − 1.20 × flatRate))`. Only computed when `ans.vatStatus === 'flatrate'`. Added to `taxableProfit` before corporation tax (it is genuine company profit subject to CT). Stored as `ltdCalcResult.frsSurplus`.
- **Dividends:** each director receives `dividendsAvailable × (shareholding / 100)`.
- **PA tapering** applies per director based on their individual `salary + dividendsReceived`.
- **Dividend band thresholds are derived from the PA, not hardcoded.** Each director's
  thresholds on **total** income are `hrThreshold = pa + 37700` and `addlThreshold = 125140`.
  Dividends stack on top of salary, so the split is:
  ```
  divInBasic  = max(0, min(salary + divs, hrThreshold)   − max(salary, pa))
  divInHigher = max(0, min(salary + divs, addlThreshold) − max(salary, hrThreshold))
  divInAdd    = max(0, (salary + divs) − max(salary, addlThreshold))
  ```
  The £500 dividend allowance is applied band-by-band from the bottom up. It is taxed at 0%
  but still consumes band, which is why it is subtracted from each band's taxable amount
  rather than shifting the higher bands down.
- **Effective rate** = `totalTaxPaid / annualGross × 100` where `totalTaxPaid = annualGross − totalNetTakeHome`.

## Tax rates live in ONE place — the `RATES` object

Every tax rate and threshold is defined once, in the `RATES` object at the very top of the
`<script>` IIFE (just above `EXP_CATS`). Student loan thresholds live in `STUDENT_LOAN_PLANS`
directly below it. **No calculation function contains a raw tax number.**

**To update for a new tax year:** change the values in `RATES` and `STUDENT_LOAN_PLANS`, update
the "Tax rates" section at the top of this file to match, and nothing else. Do not add rate
literals back into the calculation functions.

### Shared tax helpers (defined right after `RATES`)
All four calculators call these instead of doing their own tax maths:

| Helper | Does |
|---|---|
| `taperedPA(income)` | Personal allowance after the £100k taper (£1 per whole £2) |
| `incomeTaxOn(taxable, pa)` | Income tax on salary — fixed £37,700 basic band, 45% from £125,140 total income |
| `employeeNIOn(gross)` | Employee Class 1 NI (fixed thresholds — does not move with the PA) |
| `dividendTaxOn(salary, divs, pa)` | Dividend tax with band edges `pa + 37700` and `125140`, £500 allowance consumed bottom-up |
| `corporationTaxOn(profit)` | Corporation tax with marginal relief |
| `calcStudentLoan(plan, income)` | Student loan repayment via `STUDENT_LOAN_PLANS` |

### The four calculators

| Function | Route | Notes |
|---|---|---|
| `calculateIR35()` | Inside IR35 output (screen 9) | full calc incl. pension + student loan |
| `calculateIR35ForComparison()` | Comparison (screen 29) | assumes £20/wk umbrella, no pension/student loan |
| `calculateLtdCo()` | Ltd Co output (screen 19) | full calc, per-director |
| `calculateLtdCoForComparison()` | Comparison (screen 29) | single director on salary = `RATES.personalAllowance`, expenses excluded |

The comparison functions deliberately omit student loan deductions.

### Floating-point gotcha in the IR35 pot formula
`potMultiplier` must be written `1 + (RATES.employerNiRate + RATES.apprenticeshipLevy)` — **with
the inner brackets**. Without them, `1 + 0.15 + 0.005` evaluates to `1.1549999999999998`
instead of `1.155`, shifting every Inside IR35 result by a tiny amount.

### Still hardcoded outside `RATES` (UI defaults, not calculations)
The £12,570 **default salary** shown in the input boxes is still a literal in the HTML
(`value="12570"` on `ltd-salary` and `dir-salary-loop`) and in the JS that resets or pre-fills
those inputs. It is only a pre-filled suggestion — it does not affect any calculation — but if
the personal allowance ever changes, update those defaults too.

## IR35 calculation logic — critical, do not change without explicit instruction

The IR35 gross salary is back-calculated from the available pot. Never calculate Employer NI
or the Apprenticeship Levy on the gross contract value (annualGross) — they must come out of
the pot and be derived from the gross salary.

### Step-by-step formula

1. `workingDays = (52 × daysPerWeek) − holidayDays`
2. `annualGross = dayRate × workingDays` (gross contract value)
3. `umbrellaAnnual = umbrellaWeekly × 52`
4. `pot = annualGross − umbrellaAnnual` (what the umbrella passes through)
5. `grossSalary = (pot + 750) / 1.155`
   - Derived by rearranging: pot = grossSalary × 1.155 − 750
   - Where 1.155 = 1 + 0.15 (Employer NI) + 0.005 (Levy), and 750 = 0.15 × £5,000 threshold
   - In code these are computed from `RATES` as `niOffset` and `potMultiplier`, so they
     update automatically if employer NI or the levy changes (see the floating-point gotcha above)
6. `employerNI = max(0, (grossSalary − 5000) × 0.15)`
7. `levy = grossSalary × 0.005`
8. **Sense check must pass:** `grossSalary + employerNI + levy === pot`

### Income tax — the 20% band is a FIXED £37,700 of taxable income

The basic rate band is always £37,700 of taxable income. It does **not** widen when the
personal allowance tapers. HMRC fixes the band at £37,700 and stacks it on top of whatever
personal allowance the person actually has, so the higher-rate threshold is `pa + 37,700`
of total income — £50,270 at the standard PA, but only £37,700 once the PA reaches zero.

The 45% band starts at £125,140 of **total** income, which is `125140 − pa` in taxable terms.

```
basicBand = 37700                       // fixed, never varies
addlStart = max(37700, 125140 − pa)     // taxable income at which 45% begins
b1 = min(taxable, basicBand)                                        // 20%
b2 = min(max(taxable − basicBand, 0), max(addlStart − basicBand, 0)) // 40%
b3 = max(taxable − addlStart, 0)                                     // 45%
```

> **Corrected 2026-09-14.** This section previously documented `basicBand = max(0, 50270 − pa)`
> and claimed a fixed £37,700 would under-calculate tax. That was backwards: the old formula
> widened the 20% band to £50,270 once the PA tapered to zero, **under-taxing anyone earning
> over £100,000 by up to £2,514**. Verified against published HMRC figures: £100,000 → £27,432,
> £110,000 → £33,432, £125,140 → £42,516, £150,000 → £53,703.

The same `pa + 37700` threshold governs how dividends are split across bands in the Ltd Co
route — see the dividend note below. Do not reintroduce a hardcoded £50,270 in either place.

### Personal allowance tapering

```
pa = 12570
if grossSalary > 100000: pa = max(0, 12570 − floor((grossSalary − 100000) / 2))
if grossSalary ≥ 125140: pa = 0
```

### Effective rate (output screen)

Defined as: `netAnnual / annualGross × 100`
This shows the percentage of gross contract value the contractor takes home (not a deduction %).
The card label "Effective Tax Rate" uses this definition per the project spec.

### Validation test case (£700/day, 5 days, 25 holidays, £20/wk umbrella)

| Metric | Expected |
|--------|----------|
| Working days | 235 |
| Gross contract value | £164,500 |
| Umbrella fee | £1,040 |
| Available pot | £163,460 |
| Gross salary | ~£142,173 |
| Employer NI | ~£20,576 |
| Apprenticeship Levy | ~£711 |
| Sense check (gross + NI + levy) | £163,460 ✓ |
| Income tax | ~£50,181 |
| Employee NI | ~£4,854 |
| Net annual | ~£87,138 |
| Effective rate | ~52.97% |

Income tax, net annual and effective rate were **revised on 2026-09-14** when the basic-rate
band bug was fixed (previously ~£47,667 / ~£89,652 / ~54.5%). Gross salary £142,173 has a
tapered PA of £0, so this case exercises the corrected band logic directly.

## Disclaimer (always present on output screens)
"This calculator provides estimates only and does not constitute financial or tax advice.
Tax rules are complex and individual circumstances vary. Please consult a qualified accountant."

## After every task — always do this without asking
After completing any code change:
1. Update this CLAUDE.md file if anything changed about how the app works, its screen map, navigation logic, calculation rules, or build status.
2. Update README.md if it exists and the change is user-facing.
3. Commit everything to git (index.html + any updated docs) with a clear commit message. Do not ask for confirmation — just do it.
