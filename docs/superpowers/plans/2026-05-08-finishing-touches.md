# Finishing Touches — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a comparison screen, PWA installability, and final polish (button rename, ghost button) to the single-file contractor calculator.

**Architecture:** All changes are in `index.html`. CSS goes inside the existing `<style>` block. HTML goes inside `<div id="app">`. JS goes inside the existing IIFE `<script>`. No new files. No existing calculations touched.

**Tech Stack:** Vanilla JS/CSS/HTML, single file, no build tools.

---

### Task 1: Rename "Recalculate" → "Start Over" on both output screens

**Files:**
- Modify: `index.html:1319` and `index.html:1329`

- [ ] **Step 1: Make the change**

In `index.html`, find line 1319:
```javascript
9: { pct:100, back:true,  step:'Done',   btn:'Recalculate', ok:function(){ return true; } },
```
Change to:
```javascript
9: { pct:100, back:true,  step:'Done',   btn:'Start Over', ok:function(){ return true; } },
```

Find line 1329:
```javascript
19: { pct:100, back:true, step:'Done',   btn:'Recalculate', ok:function(){ return true; } },
```
Change to:
```javascript
19: { pct:100, back:true, step:'Done',   btn:'Start Over', ok:function(){ return true; } },
```

- [ ] **Step 2: Verify**

Open `index.html` in a browser. Complete the Inside IR35 flow to reach the output screen. Confirm the bottom button reads "Start Over". Do the same for the Ltd Co flow to screen 19.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: rename Recalculate to Start Over on output screens"
```

---

### Task 2: Add CSS for comparison cards, ghost button, and PWA banner

**Files:**
- Modify: `index.html` — append to existing `<style>` block (just before `</style>`)

- [ ] **Step 1: Add CSS**

Find the closing `</style>` tag (currently just after `.btn:disabled{...}` around line 280). Insert this block immediately before `</style>`:

```css
/* ── Ghost comparison button ── */
.ghost-btn{
  display:block;width:100%;height:52px;
  background:none;border:1.5px solid rgba(255,255,255,.25);border-radius:100px;
  font-family:inherit;font-size:15px;font-weight:600;
  color:rgba(255,255,255,.7);cursor:pointer;letter-spacing:-.1px;
  margin-bottom:14px;
  transition:border-color 150ms,color 150ms;
  -webkit-tap-highlight-color:transparent;
}
.ghost-btn:active{border-color:rgba(255,255,255,.6);color:var(--white)}

/* ── Comparison screen cards ── */
.comp-winner{
  background:rgba(0,196,140,.06)!important;
  border:1.5px solid var(--emerald)!important;
  position:relative;
}
.best-badge{
  position:absolute;top:14px;right:16px;
  padding:3px 10px;border-radius:100px;
  background:var(--emerald);
  font-size:11px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;
  color:#0D1B2A;
}
.comp-row{
  display:flex;justify-content:space-between;
  margin-top:12px;padding-top:12px;
  border-top:1px solid rgba(13,27,42,.08);
}
.comp-col{flex:1}
.comp-col-lbl{
  font-size:11px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;
  color:rgba(13,27,42,.4);margin-bottom:3px;
}
.comp-col-val{
  font-family:'DM Serif Display',serif;font-size:24px;color:#0D1B2A;
}
.comp-col-val.pos{color:#009E72}

/* ── PWA install banner ── */
#pwa-banner{
  position:fixed;
  bottom:calc(env(safe-area-inset-bottom,0px) + 82px);
  left:0;right:0;z-index:199;
  background:rgba(0,196,140,.12);
  border-top:1px solid rgba(0,196,140,.25);
  padding:10px 20px;
  display:none;
  align-items:center;gap:10px;
  font-size:13px;color:rgba(255,255,255,.8);
}
#pwa-banner-text{flex:1;line-height:1.4}
#pwa-dismiss{
  background:none;border:none;color:rgba(255,255,255,.5);
  font-size:18px;cursor:pointer;padding:4px;
  min-width:32px;min-height:32px;
  display:flex;align-items:center;justify-content:center;
  -webkit-tap-highlight-color:transparent;
}
```

- [ ] **Step 2: Verify**

Open DevTools → Elements. Confirm `.ghost-btn`, `.comp-winner`, `.best-badge`, and `#pwa-banner` rules exist in the computed styles. No visual change expected yet since the HTML hasn't been added.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add CSS for comparison cards, ghost button, PWA banner"
```

---

### Task 3: Add PWA head tags

**Files:**
- Modify: `index.html` — `<head>` section, after line 9 (`<meta name="mobile-web-app-capable" ...>`)

- [ ] **Step 1: Add meta tags**

Find the block of existing meta tags (lines 7–9):
```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="mobile-web-app-capable" content="yes">
```

Add two lines immediately after:
```html
<meta name="apple-mobile-web-app-title" content="Contractor Calc">
<link rel="apple-touch-icon" sizes="180x180" href="data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Crect width='180' height='180' rx='40' fill='%230D1B2A'/%3E%3Cpath d='M90 24L40 48V90C40 128 63 158 90 163C117 158 140 128 140 90V48L90 24Z' fill='%2300C48C'/%3E%3Cpath d='M68 90L80 102L112 70' stroke='%230D1B2A' stroke-width='10' stroke-linecap='round' stroke-linejoin='round'/%3E%3C%2Fsvg%3E">
```

- [ ] **Step 2: Verify**

Open DevTools → Application → Manifest (Chrome) or check `<head>` in Elements. Confirm `apple-mobile-web-app-title` meta and `apple-touch-icon` link are present. On an iPhone in Safari, go to Share → Add to Home Screen; the icon and name should appear correctly.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add PWA meta tags and apple-touch-icon"
```

---

### Task 4: Add HTML — screen 29, ghost buttons, PWA banner

**Files:**
- Modify: `index.html` — three insertion points inside `<div id="app">`

- [ ] **Step 1: Add ghost button to screen 9 (IR35 output)**

In screen 9, find the disclaimer (around line 579):
```html
    <div class="disclaimer">
      This calculator provides estimates only and does not constitute financial or tax advice.
      Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
    </div>
```

Add the ghost button immediately before it:
```html
    <button class="ghost-btn" onclick="goToComparison()">See all scenarios compared</button>
    <div class="disclaimer">
      This calculator provides estimates only and does not constitute financial or tax advice.
      Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
    </div>
```

- [ ] **Step 2: Add ghost button to screen 19 (Ltd Co output)**

In screen 19, find the disclaimer (around line 893):
```html
    <div class="disclaimer">
      This calculator provides estimates only and does not constitute financial or tax advice.
      Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
    </div>
```

Add the ghost button immediately before it:
```html
    <button class="ghost-btn" onclick="goToComparison()">See all scenarios compared</button>
    <div class="disclaimer">
      This calculator provides estimates only and does not constitute financial or tax advice.
      Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
    </div>
```

- [ ] **Step 3: Add screen 29 HTML**

Find the `<!-- EXPENSES TOTAL (fixed bar...) -->` comment (currently just before line 1239). Add screen 29 immediately before it:

```html
<!-- SCREEN 29 — Comparison -->
<div class="screen" id="screen-29">
  <div class="lbl">Compare Scenarios</div>
  <h2 class="q-h" style="margin-bottom:8px">How do the scenarios stack up?</h2>
  <p class="q-s">Based on your inputs. Estimated scenarios use default values.</p>
  <div id="comparison-cards" class="out-cards"></div>
  <div class="disclaimer" style="margin-top:14px">
    This calculator provides estimates only and does not constitute financial or tax advice.
    Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
  </div>
</div>

```

- [ ] **Step 4: Add PWA banner HTML**

Find `<!-- BOTTOM CHROME -->` (currently just before line 1242). Add the banner immediately before it:

```html
<!-- PWA INSTALL BANNER -->
<div id="pwa-banner">
  <svg width="20" height="20" viewBox="0 0 20 20" fill="none" aria-hidden="true">
    <path d="M10 2V12M10 2L7 5M10 2L13 5" stroke="white" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
    <rect x="4" y="14" width="12" height="4" rx="1" stroke="white" stroke-width="1.6"/>
  </svg>
  <span id="pwa-banner-text">Tap <strong>Share</strong> → "Add to Home Screen" to install</span>
  <button id="pwa-dismiss" onclick="dismissPWABanner()" aria-label="Dismiss">&#x2715;</button>
</div>

```

- [ ] **Step 5: Verify**

Open `index.html` in browser. Scroll to the bottom of the IR35 output screen (screen 9) and confirm the ghost "See all scenarios compared" button appears above the disclaimer. Do the same for screen 19. The button will not navigate anywhere yet. Confirm the PWA banner div exists in the DOM but is hidden (`display:none`).

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add screen 29 HTML, ghost comparison buttons, PWA banner"
```

---

### Task 5: Extend navigation JS — SCREENS[29], onContinue, staggerIn, onEnter

**Files:**
- Modify: `index.html` — SCREENS object, `onContinue`, `staggerIn`, `onEnter`

- [ ] **Step 1: Add SCREENS[29]**

Find the closing of the SCREENS object (just before line 1353 `};`). The last entry is screen 28. Add screen 29 after screen 28:

```javascript
  28: { pct: function(){return loopPct(6);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var d=ans.directors[dirLoop.current]; return !!(d&&d.studentLoan!==null&&d.studentLoan!==undefined); } },
  29: { pct:100, back:true, step:'Compare', btn:'Start Over', ok:function(){ return true; } },
```

- [ ] **Step 2: Extend onContinue to handle screen 29**

Find line 1514:
```javascript
  if (cur === 9 || cur === 19) {
```
Change to:
```javascript
  if (cur === 9 || cur === 19 || cur === 29) {
```

- [ ] **Step 3: Extend staggerIn for screen 29**

Find line 1870 (inside `staggerIn`):
```javascript
  var sel = cur === 19 ? '#screen-19' : '#screen-9';
```
Change to:
```javascript
  var sel = cur === 19 ? '#screen-19' : cur === 29 ? '#screen-29' : '#screen-9';
```

- [ ] **Step 4: Add onEnter(29) handler**

Find the `onEnter` function. The last block inside it is:
```javascript
  if (idx === 19) {
    renderLtdOutput();
    resetLtdNums();
    setTimeout(staggerIn, 100);
    setTimeout(countUp, 200);
  }
```

Add immediately after (before the closing `}` of `onEnter`):
```javascript
  if (idx === 29) {
    var s29 = buildComparisonScenarios();
    renderComparisonCards(s29);
    setTimeout(staggerIn, 100);
  }
```

- [ ] **Step 5: Verify**

Open browser. Complete the IR35 flow to screen 9. Tap "See all scenarios compared". Confirm:
- The screen slides in (screen 29)
- The header reads "Compare Scenarios"
- The bottom button reads "Start Over"
- The back button is visible and returns to screen 9
- Tapping "Start Over" resets to screen 0
- The `#comparison-cards` div is empty (rendering functions not yet added)

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: wire up screen 29 navigation — SCREENS config, onContinue, staggerIn, onEnter"
```

---

### Task 6: Add comparison calculation helper functions

**Files:**
- Modify: `index.html` — add two pure functions before `renderOutput` (currently line 1761)

- [ ] **Step 1: Add calculateIR35ForComparison**

Find `function renderOutput()` (line 1761). Insert this function immediately before it:

```javascript
function calculateIR35ForComparison(dayRate, workingDays) {
  var annualGross    = dayRate * workingDays;
  var umbrellaAnnual = 20 * 52;
  var pot            = annualGross - umbrellaAnnual;
  var employeeGross  = (pot + 750) / 1.155;
  var employerNI     = Math.max(0, (employeeGross - 5000) * 0.15);
  var levy           = employeeGross * 0.005;

  var pa = 12570;
  if (employeeGross > 100000) pa = Math.max(0, 12570 - Math.floor((employeeGross - 100000) / 2));

  var taxable   = Math.max(0, employeeGross - pa);
  var basicBand = Math.max(0, 50270 - pa);
  var b1 = Math.min(taxable, basicBand);
  var b2 = Math.min(Math.max(taxable - basicBand, 0), 74870);
  var b3 = Math.max(taxable - (basicBand + 74870), 0);
  var incomeTax = b1 * 0.20 + b2 * 0.40 + b3 * 0.45;

  var employeeNI = 0;
  if (employeeGross > 12570) {
    employeeNI += Math.min(employeeGross - 12570, 37700) * 0.08;
    if (employeeGross > 50270) employeeNI += (employeeGross - 50270) * 0.02;
  }

  var netAnnual  = employeeGross - incomeTax - employeeNI;
  var netMonthly = netAnnual / 12;
  var effRate    = annualGross > 0 ? (netAnnual / annualGross) * 100 : 0;
  return { netAnnual: netAnnual, netMonthly: netMonthly, effRate: effRate };
}

function calculateLtdCoForComparison(dayRate, workingDays) {
  var annualGross   = dayRate * workingDays;
  var salary        = 12570;
  var empNI         = Math.max(0, (salary - 5000) * 0.15);
  var taxableProfit = annualGross - salary - empNI;

  var corpTax = 0;
  if (taxableProfit > 0) {
    if (taxableProfit <= 50000) {
      corpTax = taxableProfit * 0.19;
    } else if (taxableProfit <= 250000) {
      corpTax = taxableProfit * 0.25 - (250000 - taxableProfit) * (3 / 200);
    } else {
      corpTax = taxableProfit * 0.25;
    }
  }

  var dividendsAvailable = Math.max(0, taxableProfit - corpTax);
  var divReceived        = dividendsAvailable;
  var totalIncome        = salary + divReceived;

  var pa = 12570;
  if (totalIncome > 100000) pa = Math.max(0, 12570 - Math.floor((totalIncome - 100000) / 2));

  var taxableSalary  = Math.max(0, salary - pa);
  var basicBandWidth = Math.max(0, 50270 - pa);
  var salTax = Math.min(taxableSalary, basicBandWidth) * 0.20
             + Math.min(Math.max(taxableSalary - basicBandWidth, 0), 74870) * 0.40
             + Math.max(taxableSalary - basicBandWidth - 74870, 0) * 0.45;

  var netSalary = salary - salTax;

  var divs        = divReceived;
  var divInBasic  = Math.max(0, Math.min(salary + divs, 50270)  - Math.max(salary, pa));
  var divInHigher = Math.max(0, Math.min(salary + divs, 125140) - Math.max(salary, 50270));
  var divInAdd    = Math.max(0, (salary + divs) - Math.max(salary, 125140));
  var allow       = 500;
  var txBasic     = Math.max(0, divInBasic  - allow); allow = Math.max(0, allow - divInBasic);
  var txHigher    = Math.max(0, divInHigher - allow); allow = Math.max(0, allow - divInHigher);
  var txAdd       = Math.max(0, divInAdd    - allow);
  var divTax      = txBasic * 0.1075 + txHigher * 0.3575 + txAdd * 0.3935;

  var netTakeHome = netSalary + divReceived - divTax;
  var netMonthly  = netTakeHome / 12;
  var effRate     = annualGross > 0 ? (netTakeHome / annualGross) * 100 : 0;
  return { netAnnual: netTakeHome, netMonthly: netMonthly, effRate: effRate };
}

```

- [ ] **Step 2: Sanity-check in browser console**

Open browser, complete the IR35 flow, then in DevTools console run:
```javascript
calculateIR35ForComparison(700, 235)
// Expected: netAnnual ≈ 89652, netMonthly ≈ 7471, effRate ≈ 54.5
```

Also:
```javascript
calculateLtdCoForComparison(700, 235)
// Expected: netAnnual > 89652 (Ltd Co should beat IR35 at this income level)
```

Both should return objects with numeric `netAnnual`, `netMonthly`, `effRate`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add calculateIR35ForComparison and calculateLtdCoForComparison helpers"
```

---

### Task 7: Add comparison rendering — buildComparisonScenarios, renderComparisonCards, goToComparison

**Files:**
- Modify: `index.html` — add functions after `calculateLtdCoForComparison`, before `renderOutput`

- [ ] **Step 1: Add buildComparisonScenarios, renderComparisonCards, goToComparison**

Find `function renderOutput()`. These three functions go immediately before it (directly after `calculateLtdCoForComparison`):

```javascript
function buildComparisonScenarios() {
  var origin = history.length > 0 ? history[history.length - 1] : null;
  var scenarios = [];

  if (origin === 9 && calcResult) {
    scenarios.push({
      name: 'Inside IR35',
      subtitle: '',
      netAnnual: calcResult.netAnnual,
      netMonthly: calcResult.netMonthly,
      effRate: calcResult.effRate
    });
    var dayRate1 = calcResult.workingDays > 0 ? calcResult.annualGross / calcResult.workingDays : 0;
    var ltdEst = calculateLtdCoForComparison(dayRate1, calcResult.workingDays);
    scenarios.push({
      name: 'Ltd Co',
      subtitle: 'Est. · 1 director, £12,570 salary',
      netAnnual: ltdEst.netAnnual,
      netMonthly: ltdEst.netMonthly,
      effRate: ltdEst.effRate
    });
  } else if (origin === 19 && ltdCalcResult) {
    var numDirs = +ans.numDirectors || 1;
    var dirLabel = numDirs === 1 ? '1 director' : numDirs + ' directors';
    scenarios.push({
      name: 'Ltd Co · ' + dirLabel,
      subtitle: '',
      netAnnual: ltdCalcResult.totalNetTakeHome,
      netMonthly: ltdCalcResult.netMonthly,
      effRate: ltdCalcResult.totalNetTakeHome / ltdCalcResult.annualGross * 100
    });
    var dayRate2 = ltdCalcResult.workingDays > 0 ? ltdCalcResult.annualGross / ltdCalcResult.workingDays : 0;
    var ir35Est = calculateIR35ForComparison(dayRate2, ltdCalcResult.workingDays);
    scenarios.push({
      name: 'Inside IR35',
      subtitle: 'Est. · £20/wk umbrella',
      netAnnual: ir35Est.netAnnual,
      netMonthly: ir35Est.netMonthly,
      effRate: ir35Est.effRate
    });
  }

  return scenarios;
}

function renderComparisonCards(scenarios) {
  var container = document.getElementById('comparison-cards');
  if (!container) return;
  if (!scenarios.length) {
    container.innerHTML = '<p style="color:rgba(255,255,255,.5);text-align:center;padding:32px 0">No data to compare.</p>';
    return;
  }
  var maxNet  = Math.max.apply(null, scenarios.map(function(s){ return s.netAnnual; }));
  var allSame = scenarios.every(function(s){ return s.netAnnual === maxNet; });
  container.innerHTML = scenarios.map(function(s) {
    var isWinner = !allSame && s.netAnnual === maxNet;
    return '<div class="out-card' + (isWinner ? ' comp-winner' : '') + '">' +
      (isWinner ? '<div class="best-badge">Best value</div>' : '') +
      '<div class="o-lbl">' + escHtml(s.name) + '</div>' +
      (s.subtitle ? '<div style="font-size:12px;color:rgba(13,27,42,.4);margin-bottom:6px">' + escHtml(s.subtitle) + '</div>' : '') +
      '<div class="o-val' + (isWinner ? ' pos' : '') + '">' +
        '£' + Math.round(s.netAnnual).toLocaleString('en-GB') +
      '</div>' +
      '<div class="o-sub">net annual take-home</div>' +
      '<div class="comp-row">' +
        '<div class="comp-col">' +
          '<div class="comp-col-lbl">Net Monthly</div>' +
          '<div class="comp-col-val' + (isWinner ? ' pos' : '') + '">' +
            '£' + Math.round(s.netMonthly).toLocaleString('en-GB') +
          '</div>' +
        '</div>' +
        '<div class="comp-col">' +
          '<div class="comp-col-lbl">Eff. Rate</div>' +
          '<div class="comp-col-val">' + s.effRate.toFixed(1) + '%</div>' +
        '</div>' +
      '</div>' +
    '</div>';
  }).join('');
}

window.goToComparison = function() {
  history.push(cur);
  goTo(29, 1);
};

```

- [ ] **Step 2: Verify — from IR35 output**

Complete the IR35 flow (e.g. £700/day, 5 days/wk, 25 holidays, £20 umbrella, no pension). Reach screen 9. Tap "See all scenarios compared". Confirm:
- Two white cards appear on screen 29 with stagger animation
- First card: "Inside IR35" with your actual calculated net annual
- Second card: "Ltd Co" with subtitle "Est. · 1 director, £12,570 salary"
- The card with the higher net annual has a green border and "Best value" badge
- Net Monthly and Eff. Rate rows appear in each card

- [ ] **Step 3: Verify — from Ltd Co output**

Complete the Ltd Co flow. Reach screen 19. Tap "See all scenarios compared". Confirm:
- First card: "Ltd Co · 1 director" (or however many directors) with actual result
- Second card: "Inside IR35" with subtitle "Est. · £20/wk umbrella"
- Winner highlighted in green

- [ ] **Step 4: Verify — Start Over from comparison screen**

On screen 29, tap "Start Over". Confirm it resets to screen 0 (welcome screen) with a reverse slide animation. All inputs should be cleared.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add comparison screen rendering — buildComparisonScenarios, renderComparisonCards, goToComparison"
```

---

### Task 8: Add PWA JS — manifest, service worker, dismissPWABanner, syncUI update

**Files:**
- Modify: `index.html` — inside the IIFE, just before the closing `}());`

- [ ] **Step 1: Add dismissPWABanner and syncUI extension**

Find `function syncUI()` (around line 1587). Inside `syncUI`, find the end of the function body (the closing `}` after the `exp-total-lbl` block). Add the PWA banner show/hide logic at the end of `syncUI`, immediately before its closing `}`:

```javascript
  var pwaBanner = document.getElementById('pwa-banner');
  if (pwaBanner) {
    var isIOS = /iPhone|iPad|iPod/i.test(navigator.userAgent);
    var notInstalled = !navigator.standalone;
    var notDismissed = !localStorage.getItem('pwa-dismissed');
    pwaBanner.style.display = (cur === 0 && isIOS && notInstalled && notDismissed) ? 'flex' : 'none';
  }
```

- [ ] **Step 2: Add dismissPWABanner**

Find `window.syncUI = syncUI;` (the line that exposes syncUI globally). Add immediately after it:

```javascript
window.dismissPWABanner = function() {
  localStorage.setItem('pwa-dismissed', '1');
  var banner = document.getElementById('pwa-banner');
  if (banner) banner.style.display = 'none';
};
```

- [ ] **Step 3: Add manifest and service worker registration**

Find the closing `}());` of the IIFE (the very last lines of the `<script>`). Add immediately before `}());`:

```javascript
/* ── PWA: inline manifest and service worker ── */
(function() {
  var iconSvg = "data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Crect width='180' height='180' rx='40' fill='%230D1B2A'/%3E%3Cpath d='M90 24L40 48V90C40 128 63 158 90 163C117 158 140 128 140 90V48L90 24Z' fill='%2300C48C'/%3E%3Cpath d='M68 90L80 102L112 70' stroke='%230D1B2A' stroke-width='10' stroke-linecap='round' stroke-linejoin='round'/%3E%3C%2Fsvg%3E";
  var manifestObj = {
    name: 'Contractor Calculator',
    short_name: 'Contractor Calc',
    start_url: './',
    display: 'standalone',
    background_color: '#0D1B2A',
    theme_color: '#0D1B2A',
    icons: [{ src: iconSvg, sizes: 'any', type: 'image/svg+xml' }]
  };
  var manifestBlob = new Blob([JSON.stringify(manifestObj)], { type: 'application/manifest+json' });
  var manifestURL  = URL.createObjectURL(manifestBlob);
  var manifestLink = document.createElement('link');
  manifestLink.rel  = 'manifest';
  manifestLink.href = manifestURL;
  document.head.appendChild(manifestLink);

  if ('serviceWorker' in navigator) {
    var swCode = "self.addEventListener('fetch', function(){});";
    var swBlob = new Blob([swCode], { type: 'text/javascript' });
    navigator.serviceWorker.register(URL.createObjectURL(swBlob)).catch(function(){});
  }
}());
```

- [ ] **Step 4: Verify — manifest**

Open Chrome DevTools → Application → Manifest. Confirm:
- Name: "Contractor Calculator"
- Short name: "Contractor Calc"
- Display: standalone
- Theme color and background color: #0D1B2A
- At least one icon present

- [ ] **Step 5: Verify — service worker**

DevTools → Application → Service Workers. Confirm a service worker is registered (status: activated). It may show a blob: URL — that is expected.

- [ ] **Step 6: Verify — PWA banner on iOS**

On an actual iPhone running Safari (or in Chrome with an iPhone user-agent override in DevTools → Network conditions), load the page. Confirm:
- The green banner appears at the bottom of screen 0 above the "Get Started" button
- Tapping ✕ dismisses the banner permanently (refresh to confirm it stays dismissed)
- Navigating away from screen 0 hides the banner (it reappears if you return to screen 0 and haven't dismissed it)

To test in desktop Chrome: DevTools → Network → User Agent → set to "Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X)..." and reload. Then clear localStorage first: `localStorage.removeItem('pwa-dismissed')`.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add PWA manifest, service worker, and Add to Home Screen banner"
```

---

### Task 9: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: Update screen map**

In the "Screen map — Outside IR35 Ltd Co" section, there is no screen 29 entry. Add a new section after the Director detail loop table:

```markdown
### Comparison output (screen 29)
| Index | ID | Content |
|---|---|---|
| 29 | screen-29 | Comparison — stacked IR35 vs Ltd Co cards, Start Over button |
```

- [ ] **Step 2: Update navigation notes**

In the "Navigation architecture" section, add after the `getNextScreen` description:

```
- `goToComparison()` — called by the ghost pill button on screens 9 and 19; pushes current screen to history then goes to screen 29
- Screen 29 `onContinue` calls `resetAll()` and goes to screen 0 (same as screens 9 and 19)
```

- [ ] **Step 3: Update build status**

Change Stage 6 from:
```
- [ ] Stage 6: Comparison screen, PWA install, final polish
```
To:
```
- [x] Stage 6: Comparison screen, PWA install, final polish
```

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md — screen 29, Stage 6 complete, navigation notes"
```

---

### Task 10: Final verification

- [ ] **End-to-end IR35 path**

1. Open app → complete Inside IR35 flow (£500/day, 5 days, 20 holidays, £20/wk umbrella, no pension, no student loan)
2. Reach screen 9. Button reads "Start Over". Ghost pill "See all scenarios compared" visible above disclaimer.
3. Tap "See all scenarios compared" → slides to screen 29
4. Two cards visible with stagger animation. Winner (higher net annual) has green border and "Best value" badge.
5. Tap Back → returns to screen 9
6. Tap "Start Over" → resets to screen 0

- [ ] **End-to-end Ltd Co path**

1. Complete Ltd Co flow (same day rate, 1 director, £12,570 salary, some expenses, no pension)
2. Reach screen 19. Button reads "Start Over". Ghost pill visible.
3. Tap "See all scenarios compared" → screen 29 shows "Ltd Co · 1 director" and "Inside IR35 (Est.)"
4. Tap Back → returns to screen 19

- [ ] **End-to-end multi-director Ltd Co path**

1. Complete Ltd Co flow with 2 directors
2. Screen 29 shows "Ltd Co · 2 directors" card

- [ ] **PWA checks**

On iPhone Safari:
- Add to Home Screen: icon appears as shield/check on navy background, name "Contractor Calc"
- Opening from home screen: no browser chrome (standalone mode)
- Banner visible on first open, dismissed on tap ✕, not shown again after dismiss

- [ ] **Final commit**

```bash
git add index.html CLAUDE.md
git status
git commit -m "chore: finishing touches complete — comparison screen, PWA, polish"
```
