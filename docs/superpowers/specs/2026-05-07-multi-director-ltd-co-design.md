# Multi-Director Ltd Co — Design Spec
Date: 2026-05-07

## Overview

Expand the Outside IR35 — Limited Company route to support 1–10 directors with individual shareholdings, salaries, pensions, and personal tax calculations. The 1-director flow is preserved exactly as-is; multi-director adds a per-director question loop and a redesigned output screen.

---

## Decisions

| Question | Decision |
|---|---|
| 1-director name/shareholding | Skip — flow unchanged from today |
| Pension | Per-director (in loop for 2+; screens 17–18 for 1 director as today) |
| Shareholding validation UX | Option A — live running total, Continue blocked on last director if total ≠ 100% |
| Architecture | Approach A — 6 reusable virtual screens (20–25), loop controller object |

---

## Screen Map

### Unchanged — 1 director
`13 → 14 (salary) → 15 (employed elsewhere) → 16 (expenses) → 17 (pension Y/N) → 18 (pension amount, conditional) → 19 (output)`

### Multi-director (2–10)
`13 → [20 → 21 → 22 → 23 → 24 → 25 × N directors] → 16 (expenses) → 19 (output)`

Screens 17 and 18 are not visited when `numDirectors > 1`. Pension is collected inside the loop.

### Screen 13 — How many directors?
- Cards for 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
- Picking 1 → screen 14 (today's path)
- Picking 2–10 → initialise `dirLoop`, navigate to screen 20

### New screens 20–25 (reused per director, populated dynamically)

| Index | Content | Notes |
|---|---|---|
| 20 | Director N — Name | Text input; default "Director N"; heading shows "Director N of total" |
| 21 | Director N — Shareholding % | Number input 0–100; live sub-label "Allocated X of 100%"; Continue disabled on last director if total ≠ 100% |
| 22 | Director N — Salary | £ input; default £12,570; explanation shown |
| 23 | Director N — Employed elsewhere? | Yes/No cards; red warning banner if yes; no auto-advance |
| 24 | Director N — Pension? | Yes/No cards; auto-advance if no |
| 25 | Director N — Pension amount | £ input; conditional on screen 24 |

**Progress bar** treats the director loop as a single proportional block within the overall step count (screens 10–19 + loop + 16 + 19).

**Step label** on loop screens: "Director N of total" (e.g. "Director 2 of 4").

---

## Data Model

### `ans` additions
```js
ans.directors = [];   // array of director objects, one per director
```

### Director object
```js
{
  name:              "Director 1",  // string
  shareholding:      50,            // number 0–100
  salary:            12570,         // number
  employedElsewhere: "no",          // "yes" | "no"
  pension:           "yes",         // "yes" | "no"
  pensionMonthly:    200            // number; 0 if pension === "no"
}
```

### Loop controller
```js
var dirLoop = { current: 0 };   // index of director currently being entered (0-based)
```

- Initialised to `{ current: 0 }` when entering the loop from screen 13.
- Incremented after saving each completed director.
- Loop exits to screen 16 when `dirLoop.current === numDirectors`.

### `getNextScreen()` changes
```
13  → numDirectors === "1" ? 14 : (reset dirLoop, push screen 20)
20  → 21
21  → 22
22  → 23
23  → 24
24  → ans.directors[dirLoop.current].pension === "yes" ? 25 : saveDirector() → nextOrExit()
25  → saveDirector() → nextOrExit()

saveDirector():
  writes the in-progress fields into ans.directors[dirLoop.current]
  increments dirLoop.current

nextOrExit():
  if dirLoop.current < numDirectors:
    repopulate screens 20–25 (reset all inputs to defaults, update headings/sub-labels
    with new director index, clear any warning banners)
    navigate to screen 20
  else:
    navigate to screen 16
```

**"Last director"** is defined as `dirLoop.current === numDirectors − 1`.
Shareholding validation on screen 21: Continue is disabled when this is the last director and the running total ≠ 100%. For non-last directors, Continue is enabled as long as the running total so far ≤ 100% (leaving room for remaining directors).

**Back navigation:** partial director data is preserved in `ans.directors[dirLoop.current]` so fields restore on re-entry.

---

## Calculation Logic

### Data path by director count

**1 director:** calculation reads from the existing inputs and `ans` fields — `ltd-salary`, `ans.ltdEmployedElsewhere`, `ans.ltdPension`, `ltd-pension-amount` — exactly as today. `ans.directors` is not used.

**2+ directors:** calculation reads exclusively from `ans.directors[]`. Screens 14, 15, 17, 18 are not visited, so their inputs are not read.

### Company level
```
annualGross      = dayRate × workingDays
annualExpenses   = expenses × 12
totalSalaries    = sum of all director salaries
totalPensions    = sum of (pensionMonthly × 12) per director

employerNI per director = 15% × max(0, salary − £5,000)
totalEmployerNI  = sum of per-director employer NI
employmentAllowance = numDirectors === 1 ? 0 : £10,500
netEmployerNI    = max(0, totalEmployerNI − employmentAllowance)

taxableProfit    = annualGross − annualExpenses − totalSalaries − totalPensions − netEmployerNI
corporationTax   = (existing marginal relief logic: 19% ≤50k / marginal £50k–£250k / 25% >£250k)
dividendsAvailable = max(0, taxableProfit − corporationTax)
```

### Per-director allocation
```
director.dividendsReceived = dividendsAvailable × (shareholding / 100)
```

### Per-director personal tax
Each director is calculated independently:

```
personalAllowance:
  if employedElsewhere === "yes" → 0
  else if (salary + dividendsReceived) > £100,000 → taper (reduce £1 per £2 above £100k, min 0)
  else → £12,570

taxableSalary  = max(0, salary − personalAllowance)
incomeTax      = (existing band logic: 20%/40%/45%)
employeeNI     = (existing logic: 8% on £12,570–£50,270 / 2% above)
netSalary      = salary − incomeTax − employeeNI

dividendAllowance = £500
dividendTax    = (stack dividends on top of salary, apply 10.75%/35.75%/39.35% after allowance)
netDividends   = dividendsReceived − dividendTax

director.netTakeHome = netSalary + netDividends
```

### Combined totals (2+ directors only)
```
totalNetTakeHome = sum of all director.netTakeHome
totalTaxPaid     = annualGross − totalNetTakeHome   (includes corp tax, employer NI, all personal tax)
combinedEffRate  = (annualGross − totalNetTakeHome) / annualGross × 100
```

---

## Output Layout (Screen 19)

### Hero card
- Label: "Net Annual Take-Home"
- Value: combined net across all directors (same as today for 1 director)
- Count-up animation on reveal

### Company card
```
Gross Revenue          +£xxx,xxx
Expenses              -£x,xxx
Total Salaries        -£xx,xxx
Total Pensions        -£x,xxx     (hidden if no pensions)
Employer NI           -£x,xxx     (hidden if zero — common for 2–3 directors)
Corporation Tax       -£xx,xxx
─────────────────────────────────
Dividends Available    £xx,xxx
```

### Per-director cards (one per director)
Header: `[Name] · [shareholding]%`

```
[Gross Salary £xx,xxx]  [Net Salary £xx,xxx]   ← two-column
Dividends Received        +£xx,xxx
Dividend Tax              -£x,xxx
──────────────────────────────────────────
Director Net Take-Home     £xx,xxx   (bold, large)

⚠ Employed elsewhere — figures may vary          ← red banner, only if employedElsewhere === "yes"
```

For 1 director: card header shows no name or shareholding (not collected). Layout otherwise identical.

### Combined Total card (2+ directors only — hidden for 1 director)
```
Total Net Take-Home      £xxx,xxx
Total Tax Paid           £xx,xxx
Combined Effective Rate  xx.x%
```

### Net Monthly card and Effective Rate card
- Shown for 1 director (same as today).
- Hidden for 2+ directors (Combined Total card replaces them).

### Animations
- All cards stagger in with 100ms delay between each.
- Count-up (800ms ease) runs simultaneously across all £ values in all cards.
- Effective rate displayed as plain text (no count-up), same as today.

---

## Constraints

- No external libraries or frameworks introduced.
- Single-file structure (index.html) maintained.
- Design tokens (colours, fonts, border radii, transitions) unchanged.
- Inside IR35 and Sole Trader routes untouched.
- All touch targets remain minimum 48px.
- Safe area insets maintained throughout.
- Number inputs trigger numeric keyboard on iPhone (inputmode="numeric").
