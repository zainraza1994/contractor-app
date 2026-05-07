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

For the Limited Company route, supports up to 10 directors with individual shareholding
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
- [ ] Stage 4: Limited Company, single director
- [ ] Stage 5: Limited Company, multiple directors
- [ ] Stage 6: Comparison screen, PWA install, final polish

## Navigation architecture (important for future changes)
The JavaScript uses a **history-stack navigation** — not a linear index. Key pieces:

- `SCREENS` — object map (keys 0–9) with per-screen config: progress %, step label, button text, and an `ok()` function that enables/disables Continue
- `history` — array stack; `onContinue()` pushes current index before advancing; `goBack()` pops and returns
- `getNextScreen(idx)` — returns the next screen index, handling the one branch: screen 6 (pension yes/no) → screen 7 if `ans.pension === 'yes'`, else screen 8
- `ans` object — stores all answers: `{ ir35, daysPerWeek, pension, employedElsewhere }`

**When adding Outside IR35 routes:** `getNextScreen(1)` currently returns 2 unconditionally. It will need to branch based on `ans.ir35` to route Sole Trader and Limited Company flows to their own screens.

## Screen map — Inside IR35 (screens 0–9, all built)
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
| 8 | screen-8 | Employed elsewhere? Yes/No + warning banner if yes |
| 9 | screen-9 | IR35 output — full calculated breakdown |

Outside IR35 screens (Sole Trader, Ltd Co) are not yet built. They will be added after screen-9, with `getNextScreen` updated to route to them from screen 1.

## Disclaimer (always present on output screens)
"This calculator provides estimates only and does not constitute financial or tax advice.
Tax rules are complex and individual circumstances vary. Please consult a qualified accountant."
