# Limited Company (Single Director) Route — Design Spec

**Date:** 2026-05-07  
**Stage:** 4 of 6  
**Scope:** Outside IR35 → Limited Company, single director/shareholder only

---

## Overview

Add a new question flow and output screen for contractors operating Outside IR35 via a Limited Company with a single director. The Inside IR35 route is untouched. Sole Trader route is not built — Outside IR35 routes directly to Ltd Co for now.

---

## Screen Map

New screens use indices 10–19. Existing screens 0–9 are unchanged.

| Index | ID | Content |
|---|---|---|
| 10 | screen-10 | Day rate (£ input) |
| 11 | screen-11 | Days per week (1–5 tappable cards) |
| 12 | screen-12 | Holiday days per year (number input) |
| 13 | screen-13 | Number of directors (single card: "1 director") |
| 14 | screen-14 | Director salary (£ input, default £12,570) |
| 15 | screen-15 | Employed elsewhere? (Yes/No cards + warning if Yes) |
| 16 | screen-16 | Monthly company expenses (£ input, default £150) |
| 17 | screen-17 | Company pension? (Yes/No cards) |
| 18 | screen-18 | Monthly pension amount (£ input, conditional on screen 17 = Yes) |
| 19 | screen-19 | Ltd Co output screen |

---

## Navigation Changes

**Only one change to existing routing logic:**

```
getNextScreen(1): ans.ir35 === 'outside' ? 10 : 2
```

Inside IR35 (screens 1 → 2 → ... → 9) is unchanged.

New routing within the Ltd Co flow:
- 10 → 11 → 12 → 13 → 14 → 15 → 16 → 17
- 17 → 18 if `ans.ltdPension === 'yes'`, else → 19
- 18 → 19
- 19 (output): "Recalculate" resets everything and returns to screen 0

---

## New `ans` Properties

The existing `ans` object gains these new keys (reset in `resetAll`):

| Key | Type | Set by |
|---|---|---|
| `ltdDaysPerWeek` | string (1–5) | screen-11 cards |
| `numDirectors` | string ("1") | screen-13 card |
| `ltdEmployedElsewhere` | string (yes/no) | screen-15 cards |
| `ltdPension` | string (yes/no) | screen-17 cards |

New input IDs (reset in `resetAll`):

| ID | Screen | Default |
|---|---|---|
| `ltd-day-rate` | 10 | — |
| `ltd-holiday-days` | 12 | — |
| `ltd-salary` | 14 | 12570 |
| `ltd-expenses` | 16 | 150 |
| `ltd-pension-amount` | 18 | — |

---

## Screen Content

### Screen 10 — Day Rate
- Label: `Day Rate`
- Heading: `What is your day rate?`
- Subtitle: `Your standard daily charge to clients.`
- Input: `£` prefix, id `ltd-day-rate`, numeric keyboard
- Hint: `Typical rates range from £300 to £1,200 per day`

### Screen 11 — Days Per Week
- Label: `Working Pattern`
- Heading: `How many days do you work per week?`
- Subtitle: `Your typical contracted days per week.`
- Cards: 5, 4, 3, 2, 1 (same layout as screen-3, data-group `ltdDaysPerWeek`)
- Auto-advances on selection

### Screen 12 — Holiday Days
- Label: `Time Off`
- Heading: `How many days holiday do you take per year?`
- Subtitle: `Include bank holidays if you don't work them.`
- Input: number, id `ltd-holiday-days`, no prefix
- Hint: `UK contractors typically take 20–30 days including bank holidays`

### Screen 13 — Number of Directors
- Label: `Company Setup`
- Heading: `How many directors / shareholders?`
- Subtitle: `We'll calculate each director's personal tax position.`
- Cards: single card `1 director` (data-group `numDirectors`, data-value `1`)
- Auto-advances on selection

### Screen 14 — Director Salary
- Label: `Director Salary`
- Heading: `What salary does the director take?`
- Subtitle: `£12,570 is optimal — uses your personal allowance with no Income Tax or NI`
- Input: `£` prefix, id `ltd-salary`, default value `12570`

### Screen 15 — Employed Elsewhere
- Label: `Other Income`
- Heading: `Is the director employed elsewhere?`
- Subtitle: `This can affect personal allowance and NI thresholds.`
- Cards: No / Yes (data-group `ltdEmployedElsewhere`)
- If Yes: show warning banner `ltd-employed-warning`
  - Banner text: "⚠️ The director's personal allowance and NI thresholds may already be used by their other employment. The figures shown may not reflect actual tax liability — please speak to an accountant."
- No auto-advance if Yes (user must tap Continue)

### Screen 16 — Monthly Expenses
- Label: `Company Expenses`
- Heading: `What are the monthly company expenses?`
- Subtitle: `Accountancy, phone, software, travel`
- Input: `£` prefix, id `ltd-expenses`, default value `150`

### Screen 17 — Company Pension
- Label: `Pension`
- Heading: `Does the director pay into a pension through the company?`
- Subtitle: `Company contributions reduce Corporation Tax.`
- Cards: Yes / No (data-group `ltdPension`)
- Auto-advances on selection

### Screen 18 — Monthly Pension Amount (conditional)
- Label: `Pension`
- Heading: `What is the monthly company pension contribution?`
- Subtitle: `This is paid by the company, not from the director's salary.`
- Input: `£` prefix, id `ltd-pension-amount`

### Screen 19 — Ltd Co Output
See Output Layout section below.

---

## SCREENS Config Entries

```
10: { pct:8,   back:true, step:'1 of 9', btn:'Continue',    ok: ltd-day-rate > 0 }
11: { pct:18,  back:true, step:'2 of 9', btn:'Continue',    ok: ltdDaysPerWeek !== null }
12: { pct:27,  back:true, step:'3 of 9', btn:'Continue',    ok: ltd-holiday-days >= 0 and < 365 }
13: { pct:36,  back:true, step:'4 of 9', btn:'Continue',    ok: numDirectors !== null }
14: { pct:45,  back:true, step:'5 of 9', btn:'Continue',    ok: ltd-salary >= 0 }
15: { pct:54,  back:true, step:'6 of 9', btn:'Continue',    ok: ltdEmployedElsewhere !== null }
16: { pct:63,  back:true, step:'7 of 9', btn:'Continue',    ok: ltd-expenses >= 0 }
17: { pct:72,  back:true, step:'8 of 9', btn:'Continue',    ok: ltdPension !== null }
18: { pct:82,  back:true, step:'8 of 9', btn:'Continue',    ok: ltd-pension-amount > 0 }
19: { pct:100, back:true, step:'Done',   btn:'Recalculate', ok: true }
```

---

## Calculations (2026/27 UK rates)

### Company Level

```
workingDays    = (52 × ltdDaysPerWeek) - ltdHolidayDays
annualGross    = ltdDayRate × workingDays
annualExpenses = ltdExpensesMonthly × 12
annualPension  = (ltdPension === 'yes') ? ltdPensionMonthly × 12 : 0
directorSalary = ltdSalary

taxableProfit  = annualGross - annualExpenses - directorSalary - annualPension

corporationTax:
  if taxableProfit <= 0: 0
  if taxableProfit <= 50,000: taxableProfit × 0.19
  if taxableProfit <= 250,000: taxableProfit × 0.25 − (250,000 − taxableProfit) × (3/200)
  if taxableProfit > 250,000: taxableProfit × 0.25

dividendsAvailable = max(0, taxableProfit − corporationTax)
```

### Director Level

**Personal Allowance:**
```
totalPersonalIncome = directorSalary + dividendsAvailable
pa = 12,570
if ltdEmployedElsewhere === 'yes': pa = 0
else if totalPersonalIncome > 100,000:
  pa = max(0, 12,570 − floor((totalPersonalIncome − 100,000) / 2))
```

**Salary tax:**
```
taxableSalary = max(0, directorSalary − pa)
salTax = 0
basic band (after PA): min(taxableSalary, max(0, 50,270 − pa)) at 20%
higher band: min(max(taxableSalary − max(0, 50,270 − pa), 0), 74,870) at 40%
additional: max(taxableSalary − max(0, 50,270 − pa) − 74,870, 0) at 45%
```

**Salary NI (Employee):**
```
ni = 0
if directorSalary > 12,570:
  ni += min(directorSalary − 12,570, 37,700) × 0.08
  if directorSalary > 50,270: ni += (directorSalary − 50,270) × 0.02
```
Note: if employed elsewhere, NI on salary still applies (salary is from this company).

**Dividend tax:**
```
divs = dividendsAvailable
divStart = directorSalary   // where divs sit in the income ladder

// Amount of divs in each gross income band
divInBasic    = max(0, min(directorSalary + divs, 50,270) − max(directorSalary, 12,570))
divInHigher   = max(0, min(directorSalary + divs, 125,140) − max(directorSalary, 50,270))
divInAdditional = max(0, (directorSalary + divs) − max(directorSalary, 125,140))

// Apply £500 dividend allowance (reduces taxable divs in order)
allowance = 500
taxableBasic    = max(0, divInBasic − allowance);    allowance = max(0, allowance − divInBasic)
taxableHigher   = max(0, divInHigher − allowance);   allowance = max(0, allowance − divInHigher)
taxableAdditional = max(0, divInAdditional − allowance)

divTax = taxableBasic × 0.1075 + taxableHigher × 0.3575 + taxableAdditional × 0.3935
```

**Net:**
```
netSalary  = directorSalary − salTax − salaryNI
netDivs    = divs − divTax
netAnnual  = netSalary + netDivs
netMonthly = netAnnual / 12
effRate    = annualGross > 0 ? (annualGross − netAnnual) / annualGross × 100 : 0
```

---

## Output Layout (Screen 19)

### Label + Heading
- Label: `Your Ltd Co Estimate`
- Heading: `Here's your breakdown`

### Hero card
- Label: `Net Annual Take-Home`
- Value (large green, count-up): net annual
- Sub: `{workingDays} days × £{dayRate}`

### Company Summary card (white card)
- Title label: `Company`
- Gross Revenue: annualGross (positive, green)
- Expenses: −annualExpenses (red)
- Director Salary: −directorSalary (red)
- Pension: −annualPension (red, hidden if pension=no)
- Corporation Tax: −corporationTax (red)
- Dividends Available: dividendsAvailable (green)

### Director card (white card)
- Title label: `Director`
- Gross Salary / Net Salary: two sub-values in one row
- Dividends Received: divs (green)
- Dividend Tax: −divTax (red)
- Director Net Take-Home (large, green): netSalary + netDivs

### Summary cards (white cards, stagger in)
- Net Monthly Take-Home: netMonthly (green)
- Effective Tax Rate: effRate% (neutral)

### Disclaimer
Same text as Inside IR35 output.

---

## Code changes summary

1. Add HTML for screens 10–19 after screen-9 in the DOM
2. Extend `SCREENS` config object with entries for 10–19
3. Extend `getNextScreen`: add cases 10–18; change case 1 to branch on `ans.ir35`
4. Extend `ans` object with new keys; update `resetAll` to clear them
5. Add `calculateLtdCo()` function
6. Add `renderLtdOutput()` function
7. Update `onEnter()`: add auto-focus for screens 10, 12, 14, 16, 18; add output render trigger for screen 19
8. Add `addInputListeners` calls for new inputs
9. Update `pickCard()` auto-advance exclusion to include `ltdEmployedElsewhere`
10. Update `staggerIn()` and `countUp()` to target screen-19 when on that screen
11. Update `onContinue()`: screen 19 Recalculate resets and returns to 0
