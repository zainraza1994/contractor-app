# Multi-Director Ltd Co Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand the Outside IR35 Limited Company route to support 1–10 directors with individual shareholdings, salaries, pensions, and personal tax calculations, outputting one card per director plus a combined total card.

**Architecture:** Six reusable virtual screens (20–25) are populated dynamically by a `dirLoop` controller before each visit. The 1-director flow is preserved exactly: screens 14–18 are unchanged and `ans.directors[]` is never read for that path. For 2+ directors, screens 14, 15, 17, 18 are skipped; pension moves into the per-director loop.

**Tech Stack:** Vanilla JS, vanilla CSS, single HTML file (`index.html`). No build tools. Test by opening the file in a browser via `python3 -m http.server 8080` from the project root.

---

## Reference: SVG icons used in card elements

Both SVGs below are copy-pasted into every card `.c-icon`. Keep them exactly as-is.

```html
<div class="c-icon">
  <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
  <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
</div>
```

---

## Task 1: Screen 13 — Add director count cards 2–10

**Files:** Modify `index.html` — HTML section, screen-13 div

- [ ] **Step 1: Replace the cards block inside `<!-- SCREEN 13 —`**

Find the `<div class="cards">` inside `screen-13` (currently contains only the "1 director" card). Replace the entire `<div class="cards">...</div>` block with:

```html
<div class="cards cards-num">
  <div class="card" data-group="numDirectors" data-value="1" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">1 director</div><div class="c-desc">Single director and shareholder</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="2" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">2 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="3" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">3 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="4" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">4 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="5" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">5 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="6" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">6 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="7" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">7 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="8" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">8 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="9" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">9 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
  <div class="card" data-group="numDirectors" data-value="10" onclick="pickCard(this)">
    <div class="card-body"><div class="c-title">10 directors</div></div>
    <div class="c-icon">
      <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Manual check**

Open `http://localhost:8080`, go Outside IR35, reach screen 13. Verify 10 cards appear. Tap "1 director" — it should highlight green. The Continue button should activate. Tap "2 directors" — the "1 director" deselects, "2 directors" highlights. Continue activates. (Routing won't work yet — that comes in Task 5.)

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add director count cards 2-10 on screen 13"
```

---

## Task 2: Screens 20–25 HTML

**Files:** Modify `index.html` — HTML section, insert after `<!-- SCREEN 19 -->` closing div and before `<!-- BOTTOM CHROME -->`

- [ ] **Step 1: Insert the six new screens**

Find the line `<!-- BOTTOM CHROME -->` and insert the following block immediately before it:

```html
<!-- SCREEN 20 — Director name (loop) -->
<div class="screen" id="screen-20">
  <div class="lbl" id="dir-loop-lbl-20">Director 1 of 2</div>
  <h2 class="q-h">What's the director's name?</h2>
  <p class="q-s">This label appears on the results screen.</p>
  <div class="inp-row">
    <input class="num" id="dir-name" type="text" placeholder="Director 1"
      autocomplete="off" oninput="syncUI()" style="font-size:38px">
  </div>
</div>

<!-- SCREEN 21 — Director shareholding (loop) -->
<div class="screen" id="screen-21">
  <div class="lbl" id="dir-loop-lbl-21">Director 1 of 2</div>
  <h2 class="q-h">What's their shareholding?</h2>
  <p class="q-s">Percentage of company shares this director owns.</p>
  <div class="inp-row">
    <input class="num" id="dir-shareholding" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="0" autocomplete="off" oninput="syncShareholding()">
    <span class="inp-pfx" style="padding-left:4px;padding-right:0">%</span>
  </div>
  <div class="inp-hint" id="dir-shareholding-hint">Allocated: 0 of 100%</div>
</div>

<!-- SCREEN 22 — Director salary (loop) -->
<div class="screen" id="screen-22">
  <div class="lbl" id="dir-loop-lbl-22">Director 1 of 2</div>
  <h2 class="q-h">What salary does this director take?</h2>
  <p class="q-s">£12,570 is optimal — uses the personal allowance with no Income Tax or NI.</p>
  <div class="inp-row">
    <span class="inp-pfx">£</span>
    <input class="num" id="dir-salary-loop" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="0" autocomplete="off"
      value="12570" oninput="syncUI()">
  </div>
</div>

<!-- SCREEN 23 — Director employed elsewhere (loop) -->
<div class="screen" id="screen-23">
  <div class="lbl" id="dir-loop-lbl-23">Director 1 of 2</div>
  <h2 class="q-h">Is this director employed elsewhere?</h2>
  <p class="q-s">This affects their personal allowance and NI thresholds.</p>
  <div class="cards">
    <div class="card" data-group="dirEmployedElsewhere" data-value="no" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No</div>
        <div class="c-desc">This is their only source of income</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>
    <div class="card" data-group="dirEmployedElsewhere" data-value="yes" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Yes</div>
        <div class="c-desc">They also have a PAYE job or other employment</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>
  </div>
  <div class="emp-warning" id="dir-employed-warning">
    <p>&#9888;&#xFE0F; Their personal allowance and NI thresholds may already be used by other employment. Please speak to an accountant.</p>
  </div>
</div>

<!-- SCREEN 24 — Director pension Y/N (loop) -->
<div class="screen" id="screen-24">
  <div class="lbl" id="dir-loop-lbl-24">Director 1 of 2</div>
  <h2 class="q-h">Does this director pay into a pension through the company?</h2>
  <p class="q-s">Company contributions reduce Corporation Tax.</p>
  <div class="cards">
    <div class="card" data-group="dirPension" data-value="yes" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Yes</div>
        <div class="c-desc">The company makes pension contributions</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>
    <div class="card" data-group="dirPension" data-value="no" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No</div>
        <div class="c-desc">No pension contributions for this director</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>
  </div>
</div>

<!-- SCREEN 25 — Director pension amount (loop, conditional) -->
<div class="screen" id="screen-25">
  <div class="lbl" id="dir-loop-lbl-25">Director 1 of 2</div>
  <h2 class="q-h">What's the monthly pension contribution?</h2>
  <p class="q-s">This is paid by the company, not from their salary.</p>
  <div class="inp-row">
    <span class="inp-pfx">£</span>
    <input class="num" id="dir-pension-amount-loop" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="0" autocomplete="off" oninput="syncUI()">
  </div>
</div>
```

- [ ] **Step 2: Manual check**

Reload the page. The new screens exist in the DOM but are off-screen (translateX(100%)). Open DevTools → Elements and confirm `#screen-20` through `#screen-25` exist. No visual change to the flow yet.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add screens 20-25 HTML for director detail loop"
```

---

## Task 3: Screen 19 HTML — update Company card, add director container, add Combined Total card

**Files:** Modify `index.html` — screen-19 div

- [ ] **Step 1: Update the Company card**

Find the Company `<div class="out-card">` inside screen-19 (starts with `<div class="o-lbl">Company</div>`). Replace the entire card:

```html
<div class="out-card">
  <div class="o-lbl">Company</div>
  <div class="out-row">
    <span class="out-row-lbl">Gross Revenue</span>
    <span class="out-row-val pos" id="o-ltd-gross">£0</span>
  </div>
  <div class="out-row">
    <span class="out-row-lbl">Expenses</span>
    <span class="out-row-val neg" id="o-ltd-expenses-out">-£0</span>
  </div>
  <div class="out-row">
    <span class="out-row-lbl" id="o-ltd-salary-lbl">Director Salary</span>
    <span class="out-row-val neg" id="o-ltd-salary-out">-£0</span>
  </div>
  <div class="out-row" id="o-ltd-pension-row">
    <span class="out-row-lbl" id="o-ltd-pension-lbl">Pension</span>
    <span class="out-row-val neg" id="o-ltd-pension-corp">-£0</span>
  </div>
  <div class="out-row" id="o-ltd-employer-ni-row">
    <span class="out-row-lbl">Employer NI</span>
    <span class="out-row-val neg" id="o-ltd-employer-ni">-£0</span>
  </div>
  <div class="out-row">
    <span class="out-row-lbl">Corporation Tax</span>
    <span class="out-row-val neg" id="o-ltd-corp-tax">-£0</span>
  </div>
  <div class="out-row out-row-total">
    <span class="out-row-lbl out-row-lbl-bold">Dividends Available</span>
    <span class="out-row-val pos" id="o-ltd-dividends">£0</span>
  </div>
</div>
```

- [ ] **Step 2: Replace the static Director card with a dynamic container**

Find the `<div class="out-card">` that begins with `<div class="o-lbl">Director</div>` and delete the entire card. In its place put:

```html
<div id="director-cards-container"></div>
```

- [ ] **Step 3: Add Combined Total card and update Monthly/EffRate cards**

Find the `<div class="out-card">` with `id="o-ltd-monthly-card"` — if it doesn't have that id, add it. Do the same for the effective rate card. Then add a new Combined Total card. The sequence after the director container should be:

```html
<div class="out-card" id="o-combined-card" style="display:none">
  <div class="o-lbl">Combined Total</div>
  <div class="out-row out-row-total">
    <span class="out-row-lbl out-row-lbl-bold">Total Net Take-Home</span>
    <span class="out-row-val pos lg" id="o-combined-net">£0</span>
  </div>
  <div class="out-row">
    <span class="out-row-lbl">Total Tax Paid</span>
    <span class="out-row-val neg" id="o-combined-tax">-£0</span>
  </div>
  <div class="out-row">
    <span class="out-row-lbl">Combined Effective Rate</span>
    <span class="out-row-val" id="o-combined-eff-rate" style="color:#0D1B2A">0%</span>
  </div>
</div>

<div class="out-card" id="o-ltd-monthly-card">
  <div class="o-lbl">Net Monthly Take-Home</div>
  <div class="o-val pos" id="o-ltd-net-monthly">£0</div>
  <div class="o-sub">take-home per month</div>
</div>

<div class="out-card" id="o-ltd-eff-rate-card">
  <div class="o-lbl">Effective Tax Rate</div>
  <div class="o-val" id="o-ltd-eff-rate" style="color:#0D1B2A">0%</div>
  <div class="o-sub">of gross contract value</div>
</div>
```

- [ ] **Step 4: Manual check**

Reload the page, go through the 1-director flow to screen 19. The output should look identical to before except there's no Director card (the container is empty until renderLtdOutput() is called). We'll wire that up in Task 10. The Combined Total card is hidden (`display:none`).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: update screen 19 HTML for multi-director output"
```

---

## Task 4: Data model and helper functions

**Files:** Modify `index.html` — `<script>` section

- [ ] **Step 1: Add `dirLoop` and `directors` to the `ans` initialisation**

Find:
```js
var cur = 0, busy = false;
var history = [];
var ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
            ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null };
var ltdCalcResult = null;
```

Replace with:
```js
var cur = 0, busy = false;
var history = [];
var ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
            ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
            directors: [] };
var ltdCalcResult = null;
var dirLoop = { current: 0 };
```

- [ ] **Step 2: Add helper functions**

Find the line `/* ── calculations ── */` and insert the following block immediately before it:

```js
/* ── Director loop helpers ── */

function initDirLoop() {
  dirLoop.current = 0;
  ans.directors = [];
  var total = +ans.numDirectors;
  for (var i = 0; i < total; i++) {
    ans.directors.push({
      name: 'Director ' + (i + 1),
      shareholding: 0,
      salary: 12570,
      employedElsewhere: null,
      pension: null,
      pensionMonthly: 0
    });
  }
}

function allocatedSoFar() {
  var total = 0;
  for (var i = 0; i < dirLoop.current; i++) {
    total += +ans.directors[i].shareholding || 0;
  }
  return total;
}

function syncShareholding() {
  var hint = document.getElementById('dir-shareholding-hint');
  if (hint) {
    var shEl = document.getElementById('dir-shareholding');
    var current = shEl ? (+shEl.value || 0) : 0;
    var allocated = allocatedSoFar() + current;
    hint.textContent = 'Allocated: ' + allocated + ' of 100%';
    hint.style.color = allocated > 100 ? 'var(--red)' : 'rgba(255,255,255,.3)';
  }
  syncUI();
}

function populateLoopScreens() {
  var n = dirLoop.current;
  var total = +ans.numDirectors;
  var label = 'Director ' + (n + 1) + ' of ' + total;
  [20, 21, 22, 23, 24, 25].forEach(function(i) {
    var lbl = document.getElementById('dir-loop-lbl-' + i);
    if (lbl) lbl.textContent = label;
  });
  var d = ans.directors[n];
  var nameEl = document.getElementById('dir-name');
  if (nameEl) nameEl.value = d.name || ('Director ' + (n + 1));
  var shEl = document.getElementById('dir-shareholding');
  if (shEl) shEl.value = d.shareholding || '';
  var salEl = document.getElementById('dir-salary-loop');
  if (salEl) salEl.value = d.salary != null ? d.salary : 12570;
  var paEl = document.getElementById('dir-pension-amount-loop');
  if (paEl) paEl.value = d.pensionMonthly || '';
  [].forEach.call(document.querySelectorAll('[data-group="dirEmployedElsewhere"],[data-group="dirPension"]'), function(c) {
    c.classList.remove('selected');
  });
  var ewWarn = document.getElementById('dir-employed-warning');
  if (d.employedElsewhere) {
    var ewCard = document.querySelector('[data-group="dirEmployedElsewhere"][data-value="' + d.employedElsewhere + '"]');
    if (ewCard) ewCard.classList.add('selected');
    if (ewWarn) ewWarn.style.display = d.employedElsewhere === 'yes' ? 'block' : 'none';
  } else {
    if (ewWarn) ewWarn.style.display = 'none';
  }
  if (d.pension) {
    var pCard = document.querySelector('[data-group="dirPension"][data-value="' + d.pension + '"]');
    if (pCard) pCard.classList.add('selected');
  }
  syncShareholding();
}

function saveCurrentDirector() {
  var n = dirLoop.current;
  var d = ans.directors[n];
  d.name = (document.getElementById('dir-name').value || ('Director ' + (n + 1))).trim();
  d.shareholding = +document.getElementById('dir-shareholding').value || 0;
  d.salary = +document.getElementById('dir-salary-loop').value || 0;
  d.pensionMonthly = d.pension === 'yes' ? (+document.getElementById('dir-pension-amount-loop').value || 0) : 0;
  dirLoop.current++;
}

function nextDirectorOrExit() {
  if (dirLoop.current < +ans.numDirectors) {
    return 20;
  }
  return 16;
}
```

- [ ] **Step 3: Manual check**

Open browser console. Run:
```js
ans.numDirectors = '3';
initDirLoop();
console.log(ans.directors);
```
Expected: array of 3 objects each with name "Director 1/2/3", shareholding 0, salary 12570, employedElsewhere null, pension null, pensionMonthly 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add dirLoop state and helper functions"
```

---

## Task 5: SCREENS map entries 20–25 and syncUI() dynamic support

**Files:** Modify `index.html` — `<script>` section, SCREENS object and syncUI function

- [ ] **Step 1: Add helper function `loopPct` above SCREENS declaration**

Find `var SCREENS = {` and insert immediately before it:

```js
function loopPct(fieldIdx) {
  var n = dirLoop.current;
  var total = +ans.numDirectors || 1;
  return Math.round(36 + 54 * (n * 6 + fieldIdx) / (total * 6));
}
function loopStep() {
  return 'Director ' + (dirLoop.current + 1) + ' of ' + (+ans.numDirectors);
}
```

- [ ] **Step 2: Add entries to SCREENS**

Find the closing `};` of the SCREENS object (after entry `19: { ... }`). Insert the following entries before that closing `};`:

```js
  20: { pct: function(){return loopPct(0);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var el=document.getElementById('dir-name'); return el&&el.value.trim()!==''; } },
  21: { pct: function(){return loopPct(1);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){
          var shEl=document.getElementById('dir-shareholding');
          var v=shEl?+shEl.value:0;
          if(!v||v<=0||v>100) return false;
          var isLast=dirLoop.current===+ans.numDirectors-1;
          var allocated=allocatedSoFar()+v;
          return isLast ? allocated===100 : allocated<=100;
        } },
  22: { pct: function(){return loopPct(2);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var el=document.getElementById('dir-salary-loop'); return el&&el.value!==''&&+el.value>=0; } },
  23: { pct: function(){return loopPct(3);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var d=ans.directors[dirLoop.current]; return !!(d&&d.employedElsewhere!==null); } },
  24: { pct: function(){return loopPct(4);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var d=ans.directors[dirLoop.current]; return !!(d&&d.pension!==null); } },
  25: { pct: function(){return loopPct(5);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var el=document.getElementById('dir-pension-amount-loop'); return el&&el.value!==''&&+el.value>0; } },
```

- [ ] **Step 3: Update `syncUI()` to handle function-valued pct and step**

Find the `function syncUI()` body and replace it:

```js
function syncUI() {
  var c = SCREENS[cur];
  var pct = typeof c.pct === 'function' ? c.pct() : c.pct;
  fill.style.width = pct + '%';
  backBtn.classList.toggle('show', c.back);
  stepLbl.classList.toggle('show', c.step !== null);
  if (c.step) {
    var step = typeof c.step === 'function' ? c.step() : c.step;
    stepLbl.textContent = 'Step ' + step;
  }
  mainBtn.textContent = c.btn;
  mainBtn.disabled = !c.ok();
}
```

- [ ] **Step 4: Make screen 16's pct dynamic for the 2+ director flow**

Find the SCREENS entry for screen 16:
```js
16: { pct:63, back:true, step:'7 of 9', btn:'Continue', ok:function(){ var v=document.getElementById('ltd-expenses').value; return v!==''&&+v>=0; } },
```
Replace with:
```js
16: { pct:function(){return +ans.numDirectors>1?92:63;}, back:true, step:'7 of 9', btn:'Continue', ok:function(){ var v=document.getElementById('ltd-expenses').value; return v!==''&&+v>=0; } },
```

- [ ] **Step 5: Manual check**

Open browser console. Run:
```js
ans.numDirectors='2'; initDirLoop();
cur=21;
syncUI();
```
Expected: progress bar shows ~45% (36 + 54 * 1/12 ≈ 40%). Step label shows "Step Director 1 of 2". Continue disabled (no shareholding entered).

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add SCREENS entries 20-25 with dynamic pct/step"
```

---

## Task 6: Routing — getNextScreen() and goBack()

**Files:** Modify `index.html` — `<script>` section, `getNextScreen` and `goBack` functions

- [ ] **Step 1: Replace `getNextScreen()`**

Find the entire `function getNextScreen(idx) { ... }` block and replace it:

```js
function getNextScreen(idx) {
  if (idx === 0) return 1;
  if (idx === 1) return ans.ir35 === 'outside' ? 10 : 2;
  if (idx === 2) return 3;
  if (idx === 3) return 4;
  if (idx === 4) return 5;
  if (idx === 5) return 6;
  if (idx === 6) return ans.pension === 'yes' ? 7 : 8;
  if (idx === 7) return 8;
  if (idx === 8) return 9;
  if (idx === 10) return 11;
  if (idx === 11) return 12;
  if (idx === 12) return 13;
  if (idx === 13) {
    if (ans.numDirectors === '1') return 14;
    initDirLoop();
    return 20;
  }
  if (idx === 14) return 15;
  if (idx === 15) return 16;
  if (idx === 16) return +ans.numDirectors > 1 ? 19 : (ans.ltdPension === 'yes' ? 18 : 19);
  if (idx === 17) return ans.ltdPension === 'yes' ? 18 : 19;
  if (idx === 18) return 19;
  if (idx === 20) return 21;
  if (idx === 21) return 22;
  if (idx === 22) return 23;
  if (idx === 23) return 24;
  if (idx === 24) {
    if (ans.directors[dirLoop.current] && ans.directors[dirLoop.current].pension === 'yes') return 25;
    saveCurrentDirector();
    return nextDirectorOrExit();
  }
  if (idx === 25) {
    saveCurrentDirector();
    return nextDirectorOrExit();
  }
  return null;
}
```

- [ ] **Step 2: Update `goBack()` to handle director boundary crossing**

Find `window.goBack = function(){ ... };` and replace it:

```js
window.goBack = function(){
  if (history.length === 0) return;
  var prev = history.pop();
  if (cur === 20 && (prev === 24 || prev === 25) && dirLoop.current > 0) {
    dirLoop.current--;
  }
  goTo(prev, -1);
};
```

- [ ] **Step 3: Manual check — 1-director path unchanged**

Full 1-director flow: Outside IR35 → screen 10 → 11 → 12 → 13 (pick "1 director") → 14 → 15 → 16 → 17 → (18 if pension) → 19. Verify nothing broken.

- [ ] **Step 4: Manual check — 2-director routing**

Pick "2 directors" on screen 13, tap Continue. You should arrive at screen 20 (Director 1 of 2 — Name). Tap Continue through screens 21–24. If you pick "No" pension on 24, you should skip 25 and arrive at screen 20 again for Director 2 of 2. After completing Director 2, you arrive at screen 16 (expenses), then screen 19.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add multi-director routing in getNextScreen and goBack"
```

---

## Task 7: pickCard(), addInputListeners(), and onEnter()

**Files:** Modify `index.html` — `<script>` section

- [ ] **Step 1: Update `pickCard()` to handle director loop card groups**

Find `window.pickCard = function(el) {` and replace the entire function:

```js
window.pickCard = function(el) {
  var grp = el.dataset.group;
  [].forEach.call(document.querySelectorAll('[data-group="'+grp+'"]'), function(c){
    c.classList.remove('selected');
  });
  el.classList.add('selected');
  if (grp === 'dirEmployedElsewhere') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].employedElsewhere = el.dataset.value;
  } else if (grp === 'dirPension') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].pension = el.dataset.value;
  } else {
    ans[grp] = el.dataset.value;
  }
  if (grp === 'employedElsewhere') {
    document.getElementById('employed-warning').style.display = el.dataset.value === 'yes' ? 'block' : 'none';
  }
  if (grp === 'ltdEmployedElsewhere') {
    document.getElementById('ltd-employed-warning').style.display = el.dataset.value === 'yes' ? 'block' : 'none';
  }
  if (grp === 'dirEmployedElsewhere') {
    document.getElementById('dir-employed-warning').style.display = el.dataset.value === 'yes' ? 'block' : 'none';
  }
  syncUI();
  el.style.transform = 'scale(1.025)';
  setTimeout(function(){ el.style.transform = ''; }, 160);
  var isEmpElsewhere = grp === 'employedElsewhere' || grp === 'ltdEmployedElsewhere' || grp === 'dirEmployedElsewhere';
  var autoAdvance = !isEmpElsewhere || el.dataset.value === 'no';
  if (autoAdvance) {
    setTimeout(function(){
      if (!busy && SCREENS[cur].ok()) {
        history.push(cur); goTo(getNextScreen(cur), 1);
      }
    }, 390);
  }
};
```

- [ ] **Step 2: Add input listeners for loop screen inputs**

Find the block of `addInputListeners('...')` calls and add these lines after the existing ones:

```js
addInputListeners('dir-name');
addInputListeners('dir-salary-loop');
addInputListeners('dir-pension-amount-loop');
```

For the shareholding input, the `oninput` already calls `syncShareholding()` which calls `syncUI()`. Also add Enter-key support by finding the `addInputListeners` function and noting it attaches a keydown handler. Add shareholding to the listener too:

```js
addInputListeners('dir-shareholding');
```

But note: the shareholding input calls `syncShareholding()` via `oninput`, not `syncUI()`. The `addInputListeners` function also attaches a keydown handler for Enter. That Enter handler calls `syncUI()` and then `getNextScreen()`. This is fine — the ok() validator for screen 21 already checks the value, so pressing Enter only advances when valid.

- [ ] **Step 3: Update `onEnter()` to handle screens 20–25**

Find `function onEnter(idx) {` and add the following cases inside it, after the existing `if (idx === 18)` block:

```js
  if (idx === 20) {
    populateLoopScreens();
    setTimeout(function(){ var el=document.getElementById('dir-name'); if(el) el.focus(); }, 340);
  }
  if (idx === 21) {
    syncShareholding();
    setTimeout(function(){ var el=document.getElementById('dir-shareholding'); if(el) el.focus(); }, 340);
  }
  if (idx === 22) setTimeout(function(){ var el=document.getElementById('dir-salary-loop'); if(el) el.focus(); }, 340);
  if (idx === 25) setTimeout(function(){ var el=document.getElementById('dir-pension-amount-loop'); if(el) el.focus(); }, 340);
```

- [ ] **Step 4: Manual check — director loop UX**

Go through the 2-director flow:
- Screen 20: default name "Director 1 of 2" in lbl, input pre-filled with "Director 1". Back button present.
- Screen 21: shareholding hint shows "Allocated: 0 of 100%". Type 60. Hint updates to "Allocated: 60 of 100%". Continue enabled (not last director, 60 ≤ 100).
- Go through 22, 23, 24. Pick "No" pension — skip 25, land on screen 20 for Director 2 of 2.
- Screen 21 for Director 2: type 40. Hint shows "Allocated: 100 of 100%". Continue enabled (last director, total = 100).
- Type 101. Continue disabled.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: wire pickCard, onEnter, and input listeners for director loop"
```

---

## Task 8: calculateLtdCo() — full multi-director rewrite

**Files:** Modify `index.html` — `<script>` section, `calculateLtdCo` function

- [ ] **Step 1: Replace the entire `calculateLtdCo()` function**

Find `function calculateLtdCo() {` and replace the entire function with:

```js
function calculateLtdCo() {
  var ltdDayRate         = +document.getElementById('ltd-day-rate').value || 0;
  var ltdDaysPerWeek     = +ans.ltdDaysPerWeek || 0;
  var ltdHolidayDays     = +document.getElementById('ltd-holiday-days').value || 0;
  var ltdExpensesMonthly = +document.getElementById('ltd-expenses').value || 0;
  var numDirs            = +ans.numDirectors || 1;

  var workingDays    = 52 * ltdDaysPerWeek - ltdHolidayDays;
  var annualGross    = ltdDayRate * workingDays;
  var annualExpenses = ltdExpensesMonthly * 12;

  var directors;
  if (numDirs === 1) {
    directors = [{
      name:              'Director',
      shareholding:      100,
      salary:            +document.getElementById('ltd-salary').value || 0,
      employedElsewhere: ans.ltdEmployedElsewhere || 'no',
      pension:           ans.ltdPension || 'no',
      pensionMonthly:    ans.ltdPension === 'yes' ? (+document.getElementById('ltd-pension-amount').value || 0) : 0
    }];
  } else {
    directors = ans.directors.map(function(d) {
      return {
        name:              d.name,
        shareholding:      +d.shareholding || 0,
        salary:            +d.salary || 0,
        employedElsewhere: d.employedElsewhere || 'no',
        pension:           d.pension || 'no',
        pensionMonthly:    d.pension === 'yes' ? (+d.pensionMonthly || 0) : 0
      };
    });
  }

  var totalSalaries = directors.reduce(function(s, d){ return s + d.salary; }, 0);
  var totalPensions = directors.reduce(function(s, d){ return s + d.pensionMonthly * 12; }, 0);

  var totalEmployerNI = directors.reduce(function(s, d){
    return s + Math.max(0, d.salary - 5000) * 0.15;
  }, 0);
  var employmentAllowance = numDirs === 1 ? 0 : 10500;
  var netEmployerNI = Math.max(0, totalEmployerNI - employmentAllowance);

  var taxableProfit = annualGross - annualExpenses - totalSalaries - totalPensions - netEmployerNI;

  var corporationTax = 0;
  if (taxableProfit > 0) {
    if (taxableProfit <= 50000) {
      corporationTax = taxableProfit * 0.19;
    } else if (taxableProfit <= 250000) {
      corporationTax = taxableProfit * 0.25 - (250000 - taxableProfit) * (3 / 200);
    } else {
      corporationTax = taxableProfit * 0.25;
    }
  }

  var dividendsAvailable = Math.max(0, taxableProfit - corporationTax);

  var dirResults = directors.map(function(d) {
    var divReceived = dividendsAvailable * (d.shareholding / 100);
    var totalIncome = d.salary + divReceived;

    var pa = 12570;
    if (d.employedElsewhere === 'yes') {
      pa = 0;
    } else if (totalIncome > 100000) {
      pa = Math.max(0, 12570 - Math.floor((totalIncome - 100000) / 2));
    }

    var taxableSalary  = Math.max(0, d.salary - pa);
    var basicBandWidth = Math.max(0, 50270 - pa);
    var salTax = Math.min(taxableSalary, basicBandWidth) * 0.20
               + Math.min(Math.max(taxableSalary - basicBandWidth, 0), 74870) * 0.40
               + Math.max(taxableSalary - basicBandWidth - 74870, 0) * 0.45;

    var employeeNI = 0;
    if (d.salary > 12570) {
      employeeNI += Math.min(d.salary - 12570, 37700) * 0.08;
      if (d.salary > 50270) employeeNI += (d.salary - 50270) * 0.02;
    }

    var netSalary = d.salary - salTax - employeeNI;

    var divs           = divReceived;
    var divInBasic     = Math.max(0, Math.min(d.salary + divs, 50270)  - Math.max(d.salary, pa));
    var divInHigher    = Math.max(0, Math.min(d.salary + divs, 125140) - Math.max(d.salary, 50270));
    var divInAdd       = Math.max(0, (d.salary + divs) - Math.max(d.salary, 125140));

    var allow      = 500;
    var txBasic    = Math.max(0, divInBasic  - allow); allow = Math.max(0, allow - divInBasic);
    var txHigher   = Math.max(0, divInHigher - allow); allow = Math.max(0, allow - divInHigher);
    var txAdd      = Math.max(0, divInAdd    - allow);
    var divTax     = txBasic * 0.1075 + txHigher * 0.3575 + txAdd * 0.3935;

    return {
      name:              d.name,
      shareholding:      d.shareholding,
      salary:            d.salary,
      employedElsewhere: d.employedElsewhere,
      dividendsReceived: divReceived,
      incomeTax:         salTax,
      employeeNI:        employeeNI,
      netSalary:         netSalary,
      divTax:            divTax,
      netTakeHome:       netSalary + divReceived - divTax
    };
  });

  var totalNetTakeHome = dirResults.reduce(function(s, d){ return s + d.netTakeHome; }, 0);
  var totalTaxPaid     = annualGross - totalNetTakeHome;
  var netMonthly       = totalNetTakeHome / 12;
  var effRate          = annualGross > 0 ? totalTaxPaid / annualGross * 100 : 0;

  return {
    ltdDayRate:         ltdDayRate,
    workingDays:        workingDays,
    annualGross:        annualGross,
    annualExpenses:     annualExpenses,
    totalSalaries:      totalSalaries,
    totalPensions:      totalPensions,
    netEmployerNI:      netEmployerNI,
    taxableProfit:      taxableProfit,
    corporationTax:     corporationTax,
    dividendsAvailable: dividendsAvailable,
    directors:          dirResults,
    totalNetTakeHome:   totalNetTakeHome,
    totalTaxPaid:       totalTaxPaid,
    netMonthly:         netMonthly,
    effRate:            effRate
  };
}
```

- [ ] **Step 2: Manual check — 1-director calculation**

In browser console, go through the 1-director flow, reach screen 19, open console and run:
```js
var r = calculateLtdCo();
console.log('Net take-home:', r.totalNetTakeHome, 'Corp tax:', r.corporationTax, 'Directors:', r.directors.length);
```
Expected: 1 director, netTakeHome consistent with previous values, corp tax and employer NI calculated. For a £500/day, 5 days/week, 25 holiday days, £12,570 salary, £150 expenses, no pension scenario: `annualGross` ≈ £118,750, corp tax ≈ £22,375, `totalNetTakeHome` should be in the £70k–£80k range.

- [ ] **Step 3: Manual check — 2-director calculation with Employment Allowance**

In console:
```js
ans.numDirectors = '2';
ans.directors = [
  { name:'Alice', shareholding:60, salary:12570, employedElsewhere:'no', pension:'no', pensionMonthly:0 },
  { name:'Bob',   shareholding:40, salary:12570, employedElsewhere:'no', pension:'no', pensionMonthly:0 }
];
var r = calculateLtdCo();
console.log('netEmployerNI:', r.netEmployerNI);
```
Expected: `totalEmployerNI = 2 * (12570 - 5000) * 0.15 = 2271`. After `employmentAllowance = 10500`, `netEmployerNI = max(0, 2271 - 10500) = 0`. So employer NI should be £0 for 2 directors at £12,570 salary each.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: rewrite calculateLtdCo for multi-director with employer NI and employment allowance"
```

---

## Task 9: renderLtdOutput(), countUp(), resetLtdNums(), resetAll()

**Files:** Modify `index.html` — `<script>` section

- [ ] **Step 1: Replace `renderLtdOutput()`**

Find `function renderLtdOutput() {` and replace the entire function:

```js
function renderLtdOutput() {
  ltdCalcResult = calculateLtdCo();
  var r = ltdCalcResult;
  var numDirs = +ans.numDirectors || 1;

  var daysSub = document.getElementById('o-ltd-days-sub');
  if (daysSub) daysSub.textContent = r.workingDays + ' days × £' + Math.round(r.ltdDayRate).toLocaleString('en-GB');

  var salLbl = document.getElementById('o-ltd-salary-lbl');
  if (salLbl) salLbl.textContent = numDirs === 1 ? 'Director Salary' : 'Total Salaries';
  var penLbl = document.getElementById('o-ltd-pension-lbl');
  if (penLbl) penLbl.textContent = numDirs === 1 ? 'Pension' : 'Total Pensions';

  var pensionRow = document.getElementById('o-ltd-pension-row');
  if (pensionRow) {
    var hasPension = numDirs === 1 ? ans.ltdPension === 'yes' : r.totalPensions > 0;
    pensionRow.style.display = hasPension ? '' : 'none';
  }
  var eniRow = document.getElementById('o-ltd-employer-ni-row');
  if (eniRow) eniRow.style.display = r.netEmployerNI > 0 ? '' : 'none';

  var container = document.getElementById('director-cards-container');
  if (container) {
    container.innerHTML = r.directors.map(function(d, i) {
      var header = numDirs === 1 ? 'Director' : (d.name + ' · ' + d.shareholding + '%');
      var warning = d.employedElsewhere === 'yes'
        ? '<div class="emp-warning" style="margin-top:12px"><p>⚠️ Employed elsewhere — figures may vary</p></div>'
        : '';
      return '<div class="out-card">' +
        '<div class="o-lbl">' + header + '</div>' +
        '<div class="salary-split">' +
          '<div class="salary-col"><div class="salary-col-lbl">Gross Salary</div>' +
          '<div class="salary-col-val" id="o-dir-' + i + '-gross-salary">£0</div></div>' +
          '<div class="salary-col"><div class="salary-col-lbl">Net Salary</div>' +
          '<div class="salary-col-val pos" id="o-dir-' + i + '-net-salary">£0</div></div>' +
        '</div>' +
        '<div class="out-row"><span class="out-row-lbl">Dividends Received</span>' +
          '<span class="out-row-val pos" id="o-dir-' + i + '-divs-received">£0</span></div>' +
        '<div class="out-row"><span class="out-row-lbl">Dividend Tax</span>' +
          '<span class="out-row-val neg" id="o-dir-' + i + '-div-tax">-£0</span></div>' +
        '<div class="out-row out-row-total"><span class="out-row-lbl out-row-lbl-bold">Director Net Take-Home</span>' +
          '<span class="out-row-val pos lg" id="o-dir-' + i + '-net">£0</span></div>' +
        warning +
        '</div>';
    }).join('');
  }

  var combinedCard = document.getElementById('o-combined-card');
  if (combinedCard) combinedCard.style.display = numDirs > 1 ? '' : 'none';
  if (numDirs > 1) {
    var ceff = document.getElementById('o-combined-eff-rate');
    if (ceff) ceff.textContent = r.effRate.toFixed(1) + '%';
  }

  var monthlyCard = document.getElementById('o-ltd-monthly-card');
  if (monthlyCard) monthlyCard.style.display = numDirs === 1 ? '' : 'none';
  var effRateCard = document.getElementById('o-ltd-eff-rate-card');
  if (effRateCard) effRateCard.style.display = numDirs === 1 ? '' : 'none';
  if (numDirs === 1) {
    var effEl = document.getElementById('o-ltd-eff-rate');
    if (effEl) effEl.textContent = r.effRate.toFixed(1) + '%';
  }
}
```

- [ ] **Step 2: Replace `resetLtdNums()`**

Find `function resetLtdNums() {` and replace the entire function:

```js
function resetLtdNums() {
  ['o-ltd-net-annual','o-ltd-gross','o-ltd-expenses-out','o-ltd-salary-out',
   'o-ltd-pension-corp','o-ltd-employer-ni','o-ltd-corp-tax','o-ltd-dividends',
   'o-ltd-net-monthly','o-combined-net','o-combined-tax'].forEach(function(id){
    var el = document.getElementById(id);
    if (el) el.textContent = '£0';
  });
}
```

- [ ] **Step 3: Replace the `cur === 19` branch inside `countUp()`**

Find the block starting with `if (cur === 19) {` inside `countUp()` and replace it:

```js
    if (cur === 19) {
      if (!ltdCalcResult) return;
      var lr = ltdCalcResult;
      var numDirs = +ans.numDirectors || 1;
      ITEMS = [
        { id:'o-ltd-net-annual',    val:lr.totalNetTakeHome,     neg:false },
        { id:'o-ltd-gross',         val:lr.annualGross,           neg:false },
        { id:'o-ltd-expenses-out',  val:lr.annualExpenses,        neg:true  },
        { id:'o-ltd-salary-out',    val:lr.totalSalaries,         neg:true  },
        { id:'o-ltd-pension-corp',  val:lr.totalPensions,         neg:true  },
        { id:'o-ltd-employer-ni',   val:lr.netEmployerNI,         neg:true  },
        { id:'o-ltd-corp-tax',      val:lr.corporationTax,        neg:true  },
        { id:'o-ltd-dividends',     val:lr.dividendsAvailable,    neg:false },
      ];
      lr.directors.forEach(function(d, i) {
        ITEMS.push({ id:'o-dir-'+i+'-gross-salary', val:d.salary,            neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-net-salary',   val:d.netSalary,         neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-divs-received',val:d.dividendsReceived, neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-div-tax',      val:d.divTax,            neg:true  });
        ITEMS.push({ id:'o-dir-'+i+'-net',          val:d.netTakeHome,       neg:false });
      });
      if (numDirs > 1) {
        ITEMS.push({ id:'o-combined-net', val:lr.totalNetTakeHome, neg:false });
        ITEMS.push({ id:'o-combined-tax', val:lr.totalTaxPaid,     neg:true  });
      } else {
        ITEMS.push({ id:'o-ltd-net-monthly', val:lr.netMonthly, neg:false });
      }
```

Also update the skip-if-hidden checks inside the `frame()` function in `countUp()`. Find:
```js
      var ltdPRow = document.getElementById('o-ltd-pension-row');
      if (item.id === 'o-ltd-pension-corp' && ltdPRow && ltdPRow.style.display === 'none') return;
```
And add after it:
```js
      var eniRow = document.getElementById('o-ltd-employer-ni-row');
      if (item.id === 'o-ltd-employer-ni' && eniRow && eniRow.style.display === 'none') return;
```

- [ ] **Step 4: Update `resetAll()` to clear director state**

Find `function resetAll() {` and add these two lines immediately after the `ans = { ... }` reassignment:

```js
  dirLoop = { current: 0 };
  var dirContainer = document.getElementById('director-cards-container');
  if (dirContainer) dirContainer.innerHTML = '';
  var combinedCard = document.getElementById('o-combined-card');
  if (combinedCard) combinedCard.style.display = 'none';
```

Also update the `ans` reassignment to include `directors: []`:
```js
  ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
          ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
          directors: [] };
```

- [ ] **Step 5: Manual check — 1-director output**

Complete the 1-director flow to screen 19. Verify:
- Hero shows animated total take-home
- Company card shows Gross Revenue, Expenses, Director Salary, (Pension if chosen), Employer NI row hidden (single director, no allowance — but with £12,570 salary the employer NI itself is (12570-5000)*0.15 = £1,135.50; since no employment allowance for 1 director, the employer NI row should be visible and show £1,136)
- One director card labelled "Director" (no shareholding shown)
- No Combined Total card
- Monthly take-home card visible
- Effective rate card visible
- All numbers count up from £0 over 800ms

- [ ] **Step 6: Manual check — 2-director output**

Complete a 2-director flow (Alice 60%, Bob 40%, both £12,570 salary, no pension, no employed elsewhere, £500/day, 5 days/week, 25 days holiday, £150 expenses). Reach screen 19. Verify:
- Hero shows combined take-home (Alice + Bob total)
- Company card shows Total Salaries (£25,140), Employer NI row hidden (£0 after employment allowance)
- Two director cards: "Alice · 60%" and "Bob · 40%"
- Alice's Dividends Received = 60% of dividendsAvailable; Bob's = 40%
- Combined Total card visible with correct totals
- Monthly and Effective Rate cards hidden
- All numbers animate simultaneously

- [ ] **Step 7: Manual check — employed elsewhere warning on output card**

Set one director's employed elsewhere to "yes". On screen 19, that director's card should show a red warning banner. The other director's card should not.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: wire renderLtdOutput, countUp, resetAll for multi-director output"
```

---

## Task 10: Final verification and staggerIn check

**Files:** Modify `index.html` — `<script>` section (only if staggerIn needs updating)

- [ ] **Step 1: Verify staggerIn works with dynamic cards**

`staggerIn()` already uses:
```js
var cards = [].slice.call(document.querySelectorAll(sel+' .out-card, '+sel+' .hero'));
```
Because `director-cards-container` sits inside `#screen-19`, the dynamically generated `.out-card` elements are included automatically. No change needed.

Run a 3-director flow to screen 19 and watch: hero + company card + 3 director cards + combined total card should all stagger in with 100ms delay each.

- [ ] **Step 2: Verify Recalculate resets the full loop**

On screen 19, tap "Recalculate". You should land back on screen 0. `resetAll()` should have cleared `ans.directors`, `dirLoop.current`, the director container innerHTML, and the combined card. Go through the flow again with a different director count — verify the new output is correct.

- [ ] **Step 3: Verify back navigation within the director loop**

Go through 2-director flow. On Director 2's screen 22 (salary), tap Back. You should reach Director 2's screen 21 (shareholding). Tap Back again — reach Director 2's screen 20 (name). Tap Back — you should arrive at Director 1's last screen (24 or 25 depending on pension choice) with `dirLoop.current` decremented to 0. Tap Continue — Director 1's data is re-saved (same values), and Director 2's screen 20 appears again.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: verify stagger, recalculate, and back navigation for multi-director flow"
```
