# Contractor Calculator

A UK contractor take-home pay calculator built as a mobile-first Progressive Web App (PWA).

**Live app:** [add your GitHub Pages URL here]

## What it does

Steps a UK contractor through a series of questions and calculates their net take-home pay across three scenarios:

- **Inside IR35** — via umbrella company ✅
- **Outside IR35 — Sole Trader** (coming soon)
- **Outside IR35 — Limited Company (1–3 directors)** ✅

## Tech stack

- Single `index.html` file — all HTML, CSS, and JavaScript inline
- No frameworks, no libraries, no backend
- Hosted on GitHub Pages
- Tested on iPhone Safari

## How to run locally

Open `index.html` in any browser. No build step, no server needed.

## Project structure

```
index.html          — the entire app
CLAUDE.md           — project context and instructions for Claude Code
docs/
  superpowers/
    specs/          — design documents
    plans/          — implementation plans
```

## Build status

| Stage | Feature | Status |
|-------|---------|--------|
| 1 | UI shell and design system | ✅ Done |
| 2 | Inside IR35 route and calculations | ✅ Done |
| 3 | Outside IR35 — Sole Trader | ⬜ Not started |
| 4 | Outside IR35 — Limited Company (single director) | ✅ Done |
| 5 | Outside IR35 — Limited Company (multiple directors, up to 3) | ✅ Done |
| 6 | Comparison screen, PWA install, final polish | ✅ Done |

## Tax rates used (2026/27)

- Personal Allowance: £12,570 (tapers above £100,000, eliminated above £125,140)
- Income Tax: 20% / 40% / 45%
- Employee NI: 8% on £12,570–£50,270, 2% above
- Employer NI: 15% above £5,000/year secondary threshold
- Apprenticeship Levy: 0.5% of gross salary (Inside IR35 only)
- Corporation Tax: 19% under £50k / marginal relief £50k–£250k / 25% above £250k
- Dividend Allowance: £500
- Dividend Tax: 10.75% basic / 35.75% higher / 39.35% additional
- Class 4 NI (Sole Trader): 6% on £12,570–£50,270, 2% above
- Class 2 NI: excluded (voluntary only)
- VAT Flat Rate Scheme surplus: `annualGross × (20% − 120% × flatRate%)` — added to company profit before corporation tax

## How the Inside IR35 calculation works

The key insight is that Employer NI and the Apprenticeship Levy are deducted *before* the contractor receives any salary. The gross salary must be back-calculated from the available pot:

```
working days       = (52 × days/week) − holiday days
gross contract     = day rate × working days
umbrella fee       = weekly fee × 52
available pot      = gross contract − umbrella fee

gross salary       = (pot + 750) / 1.155
employer NI        = (gross salary − £5,000) × 15%
apprenticeship levy = gross salary × 0.5%

✓ sense check: gross salary + employer NI + levy = pot
```

Income tax uses dynamic band widths that adjust when the personal allowance tapers above £100,000. Employee NI, income tax, and pension are then deducted from gross salary to arrive at net take-home.

The **Effective Rate** shown on the output screen is `net annual ÷ gross contract value × 100` — i.e. the percentage of total contract income the contractor actually keeps.

## Screen map

### Inside IR35 (screens 0–9)
| Screen | Content |
|--------|---------|
| 0 | Welcome |
| 1 | IR35 status choice |
| 2 | Day rate |
| 3 | Days per week |
| 4 | Holiday days |
| 5 | Umbrella fee |
| 6 | Pension yes/no |
| 7 | Pension % (conditional) |
| 8 | Employed elsewhere |
| 9 | Output |

### Outside IR35 — Ltd Co (screens 10–31)
| Screen | Content |
|--------|---------|
| 10 | Day rate |
| 11 | Days per week |
| 12 | Holiday days |
| 13 | Number of directors (1, 2, or 3) |
| 30 | VAT status — Not registered / Standard VAT / Flat Rate Scheme |
| 31 | VAT flat rate % — conditional, shown only for Flat Rate Scheme |
| 14 | Director salary — single director only (default £12,570) |
| 15 | Employed elsewhere — single director only |
| 27 | Student loan — single director only |
| 16 | Monthly company expenses — 9 categorised inputs; live total bar fixed at the bottom |
| 17 | Company pension yes/no — single director only |
| 18 | Monthly pension amount — single director only (conditional) |
| 19 | Output — Company card (with VAT FRS surplus, expense breakdown) + per-director cards + combined total |
| 20–28 | Director detail loop (2+ directors) — name, shareholding %, salary, employed elsewhere, pension, student loan |
| 29 | Comparison — side-by-side IR35 vs Ltd Co cards |

## Disclaimer

This calculator provides estimates only and does not constitute financial or tax advice. Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
