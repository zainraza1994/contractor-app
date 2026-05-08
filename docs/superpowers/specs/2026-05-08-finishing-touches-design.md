# Finishing Touches — Design Spec
**Date:** 2026-05-08  
**Status:** Approved

## Overview

Three finishing touches to complete Stage 6 of the contractor calculator:
1. Comparison screen — side-by-side scenario comparison from any output screen
2. PWA install — iPhone installability with "Add to Home Screen" banner
3. Final polish — button rename, currency/keyboard/viewport audit

No existing calculations or design system rules are changed.

---

## 1. Comparison Screen (screen 29)

### Trigger
A "See all scenarios compared" ghost pill button is added to the scrollable content of both output screens (9 and 19), positioned just above the existing disclaimer. It calls `goToComparison()` which pushes the current screen to `history` and calls `goTo(29, 1)`.

### Navigation
- Screen 9 → screen 29 (forward slide)
- Screen 19 → screen 29 (forward slide)
- Back button on screen 29 → standard `goBack()` returns to whichever output screen pushed it
- Bottom chrome button on screen 29 is "Start Over" — calls `resetAll()` then `goTo(0, -1)`

### SCREENS[29] config
```javascript
29: {
  pct: 100,
  back: true,
  step: 'Compare',
  btn: 'Start Over',
  ok: function() { return true; }
}
```
`onContinue()` on screen 29 calls `resetAll()` and `goTo(0, -1)` (same pattern as existing output screens).

### Scenario calculation
A function `buildComparisonScenarios()` runs in `onEnter(29)`. It returns an array of 2 scenario objects.

**Arriving from screen 9 (IR35 route):**
- Scenario A: IR35 — uses existing `calcResult` (already calculated with user inputs). Labelled "Inside IR35". No "(estimated)" tag.
- Scenario B: Ltd Co — calls a lightweight `calculateLtdCoForComparison()` with defaults: single director, salary £12,570, no expenses, no pension, no student loan. Reads day rate from `#day-rate`, working days from `calcResult.workingDays`. Labelled "Ltd Co (est.)" with a subtitle "Default: 1 director, £12,570 salary".

**Arriving from screen 19 (Ltd Co route):**
- Scenario A: Ltd Co — uses existing `ltdCalcResult`. Labelled with director count: "Ltd Co · 1 director" or "Ltd Co · N directors".
- Scenario B: IR35 — calls `calculateIR35ForComparison()` with defaults: £20/wk umbrella, no pension, no student loan. Reads day rate from `#ltd-day-rate`, working days from `ltdCalcResult.workingDays`. Labelled "IR35 (est.)" with subtitle "Default: £20/wk umbrella".

Each scenario object shape:
```javascript
{
  name: string,          // e.g. "Inside IR35"
  subtitle: string,      // e.g. "Default: £20/wk umbrella" (empty string if real data)
  netAnnual: number,
  netMonthly: number,
  effRate: number        // percentage
}
```

### Card layout
Two full-width white `.out-card` cards stacked vertically in screen 29. Each card shows:
- Small label (`.o-lbl`): scenario name
- Optional subtitle line (small, `rgba(13,27,42,.45)`): default-input caveat if applicable
- Large number (DM Serif Display, emerald if winner): net annual take-home
- Two-column row: "Net Monthly" left, "Eff. Tax Rate" right

**Winner styling:** the card with the higher `netAnnual` gets:
- `border: 1.5px solid var(--emerald)`
- `background: rgba(0,196,140,.06)`
- A "Best value" pill badge (`position: absolute`, top-right) in emerald

If both scenarios have identical `netAnnual`, neither card is highlighted.

### Comparison helper functions
`calculateIR35ForComparison(dayRate, workingDays)` — mirrors `calculateIR35()` with hardcoded defaults: umbrella £20/wk, no pension, no student loan.

`calculateLtdCoForComparison(dayRate, workingDays)` — mirrors `calculateLtdCo()` with hardcoded defaults: 1 director, salary £12,570, zero expenses, no pension, no student loan.

These are pure functions (no DOM reads) — they take day rate and working days as parameters.

### Screen 29 HTML
Standard `.screen` div with id `screen-29`. Contains:
- `.lbl` label: "Compare Scenarios"
- `.q-h` heading: "How do the scenarios stack up?"
- `.q-s` subheading: "Based on your inputs. Estimated scenarios use default values."
- `#comparison-cards` container: empty div, populated by `renderComparisonCards()`
- A full-width ghost pill "See all scenarios compared" is not needed here
- Disclaimer (same text as other output screens)

The `staggerIn()` function is extended to include `#screen-29` alongside screens 9 and 19.

---

## 2. Start Over button

**Change only:** rename `btn` value in `SCREENS[9]` and `SCREENS[19]` from `'Recalculate'` to `'Start Over'`.

`onContinue()` already handles screens 9 and 19 by calling `resetAll()` and `goTo(0, -1)`. No JS logic changes.

---

## 3. PWA Install

### New `<head>` tags
```html
<meta name="apple-mobile-web-app-title" content="Contractor Calc">
<link rel="apple-touch-icon" href="[inline SVG data URI of the shield/check badge]">
<link rel="manifest" href="data:application/manifest+json,[URL-encoded JSON]">
```

Manifest JSON:
```json
{
  "name": "Contractor Calculator",
  "short_name": "Contractor Calc",
  "start_url": "./",
  "display": "standalone",
  "background_color": "#0D1B2A",
  "theme_color": "#0D1B2A",
  "icons": [{ "src": "[SVG data URI]", "sizes": "any", "type": "image/svg+xml" }]
}
```

### Inline service worker
Registered in the IIFE at the bottom of `<script>`:
```javascript
if ('serviceWorker' in navigator) {
  var swCode = "self.addEventListener('fetch', function(){});";
  var swBlob = new Blob([swCode], {type: 'text/javascript'});
  navigator.serviceWorker.register(URL.createObjectURL(swBlob));
}
```
Minimal SW with a fetch listener (required by Chrome install criteria). Does not implement caching — single-file constraint prevents scoped offline caching via blob URL.

### "Add to Home Screen" banner
Shown only when **all** of:
- User agent matches `/iPhone|iPad|iPod/i`
- `navigator.standalone` is `false` (not already installed)
- `localStorage.getItem('pwa-dismissed')` is falsy

Position: fixed bar above the bottom chrome on screen 0 only. Hidden on all other screens.

HTML: a `<div id="pwa-banner">` placed just before `#chrome-bot`:
```
[ share-icon ]  Tap Share → "Add to Home Screen" to install   [ × ]
```

CSS:
- `position: fixed`, `bottom: calc(82px + env(safe-area-inset-bottom))`, `left: 0`, `right: 0`
- `background: rgba(0,196,140,.12)`, `border-top: 1px solid rgba(0,196,140,.25)`
- `padding: 10px 20px`, `display: flex`, `align-items: center`, `gap: 10px`
- Font size 13px, color `rgba(255,255,255,.8)`
- `z-index: 199`

Dismiss button (✕): calls `dismissPWABanner()` which sets `localStorage.setItem('pwa-dismissed','1')` and hides the element.

`syncUI()` is updated to show/hide `#pwa-banner` based on whether `cur === 0`.

---

## 4. Final Polish audit

| Item | Status | Action |
|---|---|---|
| Disclaimer on output screens | Already present on screens 9 and 19 | No change |
| Currency £ prefix on inputs | All currency inputs already have `.inp-pfx` span | No change |
| Comma formatting on inputs | `type="number"` inputs cannot show commas (correct) | No change |
| Numeric keyboard on iPhone | All inputs already have `inputmode="numeric" pattern="[0-9]*"` | No change |
| Safe area insets | Already applied throughout | No change |
| Viewport / no scrollbars | Question screens fit; output + expense screens scroll by design | No change |

All items already correct. Only the new screen 29 and PWA additions are net-new code.

---

## Screen map update

Screen 29 added:

| Index | ID | Content |
|---|---|---|
| 29 | screen-29 | Comparison output — stacked scenario cards + Start Over |

`getNextScreen()` does not need updating — screen 29 is reached only via `goToComparison()` (direct `goTo` call), not via the standard continue flow.

---

## Files changed

- `index.html` — all changes inline:
  - `<head>`: 3 new meta/link tags
  - CSS: comparison card styles, ghost button style, PWA banner style
  - HTML: `#screen-29`, `#pwa-banner` div, ghost "See all scenarios compared" buttons in screens 9 and 19
  - JS: `SCREENS[29]`, `goToComparison()`, `buildComparisonScenarios()`, `calculateIR35ForComparison()`, `calculateLtdCoForComparison()`, `renderComparisonCards()`, `dismissPWABanner()`, SW registration, banner show/hide logic in `syncUI()`, `staggerIn()` extension, rename Recalculate→Start Over in SCREENS[9] and SCREENS[19]
- `CLAUDE.md` — update screen map with screen 29, update build status Stage 6 as in-progress
