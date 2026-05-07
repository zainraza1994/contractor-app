# Contractor Calculator

A UK contractor take-home pay calculator built as a mobile-first Progressive Web App (PWA).

**Live app:** [add your GitHub Pages URL here]

## What it does

Steps a UK contractor through a series of questions and calculates their net take-home pay across three scenarios:

- **Inside IR35** — via umbrella company ✅
- **Outside IR35 — Sole Trader** (coming soon)
- **Outside IR35 — Limited Company (single director)** ✅

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
| 5 | Outside IR35 — Limited Company (multiple directors) | ⬜ Not started |
| 6 | Comparison screen, PWA install, final polish | ⬜ Not started |

## Tax rates used (2026/27)

- Personal Allowance: £12,570 (tapers above £100,000)
- Income Tax: 20% / 40% / 45%
- Employee NI: 8% to £50,270, 2% above
- Employer NI: 15% above £5,000
- Apprenticeship Levy: 0.5% of gross (Inside IR35 only)
- Corporation Tax: 19% / marginal relief / 25%
- Dividend Tax: 10.75% / 35.75% / 39.35%
- Class 4 NI (Sole Trader): 6% to £50,270, 2% above

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

### Outside IR35 — Ltd Co single director (screens 10–19)
| Screen | Content |
|--------|---------|
| 10 | Day rate |
| 11 | Days per week |
| 12 | Holiday days |
| 13 | Number of directors |
| 14 | Director salary (default £12,570) |
| 15 | Employed elsewhere |
| 16 | Monthly company expenses (default £150) |
| 17 | Company pension yes/no |
| 18 | Monthly pension amount (conditional) |
| 19 | Output — Company breakdown + Director breakdown |

## Disclaimer

This calculator provides estimates only and does not constitute financial or tax advice. Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
