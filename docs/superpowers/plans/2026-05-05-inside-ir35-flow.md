# Inside IR35 Flow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add the full Inside IR35 question flow (6 new screens) and calculated output to index.html, branching from the existing IR35 choice screen.

**Architecture:** Replace the existing linear `CFG` array navigation with a history-stack approach that supports branching (pension % screen is conditional). All changes are confined to `index.html`. Screens 0–2 remain; screens previously numbered 3–4 are renamed to 6 and 9, and new screens 3–5, 7–8 are inserted between them.

**Tech Stack:** Vanilla HTML/CSS/JS, single file, no build tools. Verification is manual browser testing.

---

## Screen Map (final)

| Index | ID        | Content                        | Notes                  |
|-------|-----------|--------------------------------|------------------------|
| 0     | screen-0  | Welcome                        | unchanged              |
| 1     | screen-1  | IR35 choice                    | unchanged              |
| 2     | screen-2  | Day rate                       | unchanged              |
| 3     | screen-3  | Days per week (cards 1–5)      | NEW                    |
| 4     | screen-4  | Days holiday (number input)    | NEW                    |
| 5     | screen-5  | Umbrella fee (£, default £20)  | NEW                    |
| 6     | screen-6  | Pension? Yes/No                | renamed from screen-3  |
| 7     | screen-7  | Pension % (conditional)        | NEW                    |
| 8     | screen-8  | Employed elsewhere? Yes/No     | NEW                    |
| 9     | screen-9  | IR35 Output                    | replaces screen-4      |

`getNextScreen(6)` returns 7 if `ans.pension === 'yes'`, else 8.

---

## Task 1: Navigation architecture

**File:** `index.html` — replace the `<script>` block content

- [ ] **Step 1: Locate and replace the JS block**

Find the opening `(function(){` on line ~345 and replace the entire script content with the following. Keep `</script></body></html>` at the end.

```javascript
(function(){
'use strict';

var cur = 0, busy = false;
var history = [];
var ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null };

var screens = [].slice.call(document.querySelectorAll('.screen'));
var fill    = document.getElementById('prog-fill');
var backBtn = document.getElementById('back-btn');
var stepLbl = document.getElementById('step-lbl');
var mainBtn = document.getElementById('main-btn');

var SCREENS = {
  0: { pct:0,   back:false, step:null,     btn:'Get Started', ok:function(){ return true; } },
  1: { pct:12,  back:true,  step:'1 of 7', btn:'Continue',    ok:function(){ return ans.ir35 !== null; } },
  2: { pct:24,  back:true,  step:'2 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('day-rate').value; return v!==''&&+v>0; } },
  3: { pct:36,  back:true,  step:'3 of 7', btn:'Continue',    ok:function(){ return ans.daysPerWeek !== null; } },
  4: { pct:48,  back:true,  step:'4 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('holiday-days').value; return v!==''&&+v>=0&&+v<365; } },
  5: { pct:60,  back:true,  step:'5 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('umbrella-fee').value; return v!==''&&+v>=0; } },
  6: { pct:72,  back:true,  step:'6 of 7', btn:'Continue',    ok:function(){ return ans.pension !== null; } },
  7: { pct:80,  back:true,  step:'6 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('pension-pct').value; return v!==''&&+v>0&&+v<=100; } },
  8: { pct:88,  back:true,  step:'7 of 7', btn:'Continue',    ok:function(){ return ans.employedElsewhere !== null; } },
  9: { pct:100, back:true,  step:'Done',   btn:'Recalculate', ok:function(){ return true; } },
};

function getNextScreen(idx) {
  if (idx === 0) return 1;
  if (idx === 1) return 2;
  if (idx === 2) return 3;
  if (idx === 3) return 4;
  if (idx === 4) return 5;
  if (idx === 5) return 6;
  if (idx === 6) return ans.pension === 'yes' ? 7 : 8;
  if (idx === 7) return 8;
  if (idx === 8) return 9;
  return null;
}

screens.forEach(function(s,i){
  s.style.transform = i===0 ? 'translateX(0)' : 'translateX(100%)';
});
syncUI();

function goTo(idx, dir) {
  if (busy || idx<0 || idx>=screens.length || idx===cur) return;
  busy = true;
  var from = screens[cur];
  var to   = screens[idx];
  to.style.transition = 'none';
  to.style.transform  = dir>0 ? 'translateX(100%)' : 'translateX(-100%)';
  void to.offsetWidth;
  to.style.transition = '';
  from.style.transform = dir>0 ? 'translateX(-100%)' : 'translateX(100%)';
  to.style.transform   = 'translateX(0)';
  cur = idx;
  syncUI();
  onEnter(idx);
  setTimeout(function(){
    from.style.transition = 'none';
    from.style.transform  = 'translateX(100%)';
    void from.offsetWidth;
    from.style.transition = '';
    busy = false;
  }, 350);
}

function onEnter(idx) {
  if (idx === 2) setTimeout(function(){ document.getElementById('day-rate').focus(); }, 340);
  if (idx === 4) setTimeout(function(){ document.getElementById('holiday-days').focus(); }, 340);
  if (idx === 5) setTimeout(function(){ document.getElementById('umbrella-fee').focus(); }, 340);
  if (idx === 7) setTimeout(function(){ document.getElementById('pension-pct').focus(); }, 340);
  if (idx === 9) {
    renderOutput();
    resetNums();
    setTimeout(staggerIn, 100);
    setTimeout(countUp, 200);
  }
}

window.goBack = function(){
  if (history.length === 0) return;
  var prev = history.pop();
  goTo(prev, -1);
};

window.onContinue = function(){
  if (!SCREENS[cur].ok()) return;
  if (cur === 9) {
    resetAll();
    goTo(0, -1);
  } else {
    var next = getNextScreen(cur);
    if (next !== null) { history.push(cur); goTo(next, 1); }
  }
};

window.pickCard = function(el) {
  var grp = el.dataset.group;
  [].forEach.call(document.querySelectorAll('[data-group="'+grp+'"]'), function(c){
    c.classList.remove('selected');
  });
  el.classList.add('selected');
  ans[grp] = el.dataset.value;
  if (grp === 'employedElsewhere') {
    document.getElementById('employed-warning').style.display =
      el.dataset.value === 'yes' ? 'block' : 'none';
  }
  syncUI();
  el.style.transform = 'scale(1.025)';
  setTimeout(function(){ el.style.transform = ''; }, 160);
  var autoAdvance = grp !== 'employedElsewhere' || el.dataset.value === 'no';
  if (autoAdvance) {
    setTimeout(function(){
      if (!busy && SCREENS[cur].ok()) {
        history.push(cur);
        goTo(getNextScreen(cur), 1);
      }
    }, 390);
  }
};

window.syncUI = syncUI;

function addInputListeners(id) {
  var el = document.getElementById(id);
  if (!el) return;
  el.addEventListener('input', syncUI);
  el.addEventListener('keydown', function(e){
    if (e.key==='Enter' && SCREENS[cur].ok()) {
      history.push(cur); goTo(getNextScreen(cur), 1);
    }
  });
}
addInputListeners('day-rate');
addInputListeners('holiday-days');
addInputListeners('umbrella-fee');
addInputListeners('pension-pct');

function syncUI() {
  var c = SCREENS[cur];
  fill.style.width = c.pct+'%';
  backBtn.classList.toggle('show', c.back);
  stepLbl.classList.toggle('show', c.step!==null);
  if (c.step) stepLbl.textContent = 'Step '+c.step;
  mainBtn.textContent = c.btn;
  mainBtn.disabled = !c.ok();
}

/* ── calculations ── */
var calcResult = null;

function calculateIR35() {
  var dayRate          = +document.getElementById('day-rate').value || 0;
  var daysPerWeek      = +ans.daysPerWeek || 0;
  var holidayDays      = +document.getElementById('holiday-days').value || 0;
  var umbrellaWeekly   = +document.getElementById('umbrella-fee').value || 0;
  var pensionPct       = ans.pension === 'yes' ? (+document.getElementById('pension-pct').value || 0) : 0;

  var workingDays      = (52 * daysPerWeek) - holidayDays;
  var annualGross      = dayRate * workingDays;
  var umbrellaAnnual   = umbrellaWeekly * 52;
  var employerNI       = Math.max(0, (annualGross - 5000) * 0.15);
  var levy             = annualGross * 0.005;
  var employeeGross    = annualGross - umbrellaAnnual - employerNI - levy;

  var pa = 12570;
  if (employeeGross > 100000) pa = Math.max(0, 12570 - Math.floor((employeeGross - 100000) / 2));

  var taxable = Math.max(0, employeeGross - pa);
  var incomeTax = 0;
  var b1 = Math.min(taxable, 37700);
  var b2 = Math.min(Math.max(taxable - 37700, 0), 74870);
  var b3 = Math.max(taxable - 112570, 0);
  incomeTax = b1 * 0.20 + b2 * 0.40 + b3 * 0.45;

  var employeeNI = 0;
  if (employeeGross > 12570) {
    employeeNI += Math.min(employeeGross - 12570, 37700) * 0.08;
    if (employeeGross > 50270) employeeNI += (employeeGross - 50270) * 0.02;
  }

  var pension    = employeeGross * (pensionPct / 100);
  var netAnnual  = employeeGross - incomeTax - employeeNI - pension;
  var netMonthly = netAnnual / 12;
  var totalDeds  = umbrellaAnnual + employerNI + levy + incomeTax + employeeNI + pension;
  var effRate    = annualGross > 0 ? (totalDeds / annualGross) * 100 : 0;

  return { annualGross:annualGross, workingDays:workingDays, umbrellaAnnual:umbrellaAnnual,
           employerNI:employerNI, levy:levy, employeeGross:employeeGross,
           incomeTax:incomeTax, employeeNI:employeeNI, pension:pension,
           netAnnual:netAnnual, netMonthly:netMonthly, effRate:effRate };
}

function renderOutput() {
  calcResult = calculateIR35();
  var r = calcResult;
  var dr = +document.getElementById('day-rate').value;
  document.getElementById('o-days-sub').textContent =
    r.workingDays + ' days × £' + Math.round(dr).toLocaleString('en-GB');
  var pensionRow = document.getElementById('o-pension-row');
  pensionRow.style.display = ans.pension === 'yes' ? '' : 'none';
  if (ans.pension === 'yes') {
    document.getElementById('o-pension-sub').textContent =
      document.getElementById('pension-pct').value + '% of employee gross';
  }
  document.getElementById('o-eff-rate').textContent = r.effRate.toFixed(1) + '%';
}

/* ── count-up ── */
function resetNums() {
  ['o-gross','o-umbrella','o-employer-ni','o-levy','o-employee-ni',
   'o-income-tax','o-pension','o-net-annual','o-net-monthly'].forEach(function(id){
    var el = document.getElementById(id);
    if (el) el.textContent = '£0';
  });
}

function countUp() {
  if (!calcResult) return;
  var r = calcResult;
  var ITEMS = [
    { id:'o-gross',       val:r.annualGross,    neg:false },
    { id:'o-umbrella',    val:r.umbrellaAnnual, neg:true  },
    { id:'o-employer-ni', val:r.employerNI,     neg:true  },
    { id:'o-levy',        val:r.levy,           neg:true  },
    { id:'o-employee-ni', val:r.employeeNI,     neg:true  },
    { id:'o-income-tax',  val:r.incomeTax,      neg:true  },
    { id:'o-pension',     val:r.pension,        neg:true  },
    { id:'o-net-annual',  val:r.netAnnual,      neg:false },
    { id:'o-net-monthly', val:r.netMonthly,     neg:false },
  ];
  var DUR = 800, t0 = null;
  function ease(t){ return 1-Math.pow(1-t,3); }
  function frame(ts) {
    if (!t0) t0 = ts;
    var p = Math.min((ts-t0)/DUR, 1);
    var e = ease(p);
    ITEMS.forEach(function(item){
      var el = document.getElementById(item.id);
      if (!el) return;
      var row = el.closest ? el.closest('[id$="-row"]') : null;
      if (row && row.style.display === 'none') return;
      el.textContent = (item.neg?'-':'')+'£'+Math.round(e*item.val).toLocaleString('en-GB');
    });
    if (p < 1) requestAnimationFrame(frame);
  }
  requestAnimationFrame(frame);
}

function staggerIn() {
  var cards = [].slice.call(document.querySelectorAll('#screen-9 .out-card, #screen-9 .hero'));
  var visible = cards.filter(function(c){ return c.style.display !== 'none'; });
  visible.forEach(function(card, i){
    card.style.opacity = '0';
    card.style.transform = 'translateY(16px)';
    card.style.transition = 'none';
    setTimeout(function(){
      card.style.transition = 'opacity 300ms ease, transform 300ms ease';
      card.style.opacity = '1';
      card.style.transform = 'translateY(0)';
    }, i * 100);
  });
}

function resetAll() {
  ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null };
  history = []; calcResult = null;
  ['day-rate','holiday-days','pension-pct'].forEach(function(id){
    var el = document.getElementById(id);
    if (el) el.value = '';
  });
  document.getElementById('umbrella-fee').value = '20';
  document.getElementById('employed-warning').style.display = 'none';
  [].forEach.call(document.querySelectorAll('.card'), function(c){ c.classList.remove('selected'); });
}

}());
```

- [ ] **Step 2: Open `index.html` in a browser. Verify:**
  - Welcome screen loads correctly
  - "Get Started" advances to IR35 choice
  - Selecting Inside/Outside IR35 auto-advances to Day Rate
  - Day Rate input accepts numbers and Continue is enabled
  - Back button returns to previous screen
  - Progress bar moves forward and backward correctly
  - Console shows no errors

---

## Task 2: New question screens (HTML)

**File:** `index.html` — HTML section, after `</div><!-- screen-2 -->` through the output screen.

- [ ] **Step 1: Add CSS for new UI elements**

Inside the `<style>` block, after the `.inp-hint` rule (line ~159), add:

```css
/* ── Compact numeric cards ── */
.cards-num .card{min-height:64px;padding:14px 20px}
.cards-num .c-title{font-size:20px;font-family:'DM Serif Display',serif}

/* ── Employed elsewhere warning ── */
.emp-warning{
  display:none;margin-top:16px;
  padding:16px;border-radius:12px;
  background:rgba(255,107,107,.1);
  border:1px solid rgba(255,107,107,.3);
}
.emp-warning p{font-size:14px;color:var(--red);line-height:1.55}

/* ── Employer NI highlight ── */
.ni-highlight{border-color:rgba(255,107,107,.35)!important;background:#fff!important}
.ni-badge{
  display:inline-block;margin-left:8px;
  padding:2px 8px;border-radius:100px;
  font-size:10px;font-weight:700;letter-spacing:.06em;text-transform:uppercase;
  background:rgba(255,107,107,.15);color:var(--red);
  vertical-align:middle;
}

/* ── Disclaimer ── */
.disclaimer{
  margin-top:8px;padding:16px 20px;
  font-size:12px;color:rgba(13,27,42,.4);line-height:1.6;
  background:#f4f5f6;border-radius:var(--r);
}
```

- [ ] **Step 2: Rename the existing pension screen from `screen-3` to `screen-6`**

Change `id="screen-3"` → `id="screen-6"` in the HTML. Also update the two `data-group="pension"` cards — they stay as-is (group name `pension` is fine).

- [ ] **Step 3: Rename the existing output screen from `screen-4` to `screen-9` and replace its content**

Replace the entire `<div class="screen" id="screen-4">...</div>` block with:

```html
<!-- SCREEN 9 — IR35 Output -->
<div class="screen" id="screen-9">
  <div class="lbl">Your IR35 Estimate</div>
  <h2 class="q-h" style="margin-bottom:20px">Here's your breakdown</h2>

  <div class="hero">
    <div class="h-lbl">Net Annual Take-Home</div>
    <div class="h-val" id="o-net-annual">£0</div>
    <div class="h-sub" id="o-days-sub">after all deductions</div>
  </div>

  <div class="out-cards">

    <div class="out-card">
      <div class="o-lbl">Net Monthly</div>
      <div class="o-val pos" id="o-net-monthly">£0</div>
      <div class="o-sub">take-home per month</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Effective Tax Rate</div>
      <div class="o-val" id="o-eff-rate" style="color:#0D1B2A">0%</div>
      <div class="o-sub">of gross contract value</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Gross Contract Value</div>
      <div class="o-val pos" id="o-gross">£0</div>
      <div class="o-sub">annual contract income</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Umbrella Company Fee</div>
      <div class="o-val neg" id="o-umbrella">-£0</div>
      <div class="o-sub">weekly fee × 52</div>
    </div>

    <div class="out-card ni-highlight">
      <div class="o-lbl">Employer NI <span class="ni-badge">Never reaches you</span></div>
      <div class="o-val neg" id="o-employer-ni">-£0</div>
      <div class="o-sub">15% above £5,000 — paid by umbrella before you see a penny</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Apprenticeship Levy</div>
      <div class="o-val neg" id="o-levy">-£0</div>
      <div class="o-sub">0.5% of gross</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Employee NI</div>
      <div class="o-val neg" id="o-employee-ni">-£0</div>
      <div class="o-sub">8% to £50,270 then 2%</div>
    </div>

    <div class="out-card">
      <div class="o-lbl">Income Tax</div>
      <div class="o-val neg" id="o-income-tax">-£0</div>
      <div class="o-sub">2026/27 rates, personal allowance applied</div>
    </div>

    <div class="out-card" id="o-pension-row">
      <div class="o-lbl">Pension Contribution</div>
      <div class="o-val neg" id="o-pension">-£0</div>
      <div class="o-sub" id="o-pension-sub"></div>
    </div>

    <div class="disclaimer">
      This calculator provides estimates only and does not constitute financial or tax advice.
      Tax rules are complex and individual circumstances vary. Please consult a qualified accountant.
    </div>

  </div>
</div>
```

- [ ] **Step 4: Add new question screens 3–5 and 7–8 into the HTML**

Insert these between the closing of `screen-2` and the opening of `screen-6`:

```html
<!-- SCREEN 3 — Days per week -->
<div class="screen" id="screen-3">
  <div class="lbl">Working Pattern</div>
  <h2 class="q-h">How many days do you work per week?</h2>
  <p class="q-s">Your typical contracted days per week.</p>
  <div class="cards cards-num">

    <div class="card" data-group="daysPerWeek" data-value="5" onclick="pickCard(this)">
      <div class="card-body"><div class="c-title">5 days</div></div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="daysPerWeek" data-value="4" onclick="pickCard(this)">
      <div class="card-body"><div class="c-title">4 days</div></div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="daysPerWeek" data-value="3" onclick="pickCard(this)">
      <div class="card-body"><div class="c-title">3 days</div></div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="daysPerWeek" data-value="2" onclick="pickCard(this)">
      <div class="card-body"><div class="c-title">2 days</div></div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="daysPerWeek" data-value="1" onclick="pickCard(this)">
      <div class="card-body"><div class="c-title">1 day</div></div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

  </div>
</div>

<!-- SCREEN 4 — Holiday days -->
<div class="screen" id="screen-4">
  <div class="lbl">Time Off</div>
  <h2 class="q-h">How many days holiday do you take per year?</h2>
  <p class="q-s">Include bank holidays if you don't work them.</p>
  <div class="inp-row">
    <input class="num" id="holiday-days" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="0" autocomplete="off" oninput="syncUI()">
  </div>
  <div class="inp-hint">UK contractors typically take 20–30 days including bank holidays</div>
</div>

<!-- SCREEN 5 — Umbrella fee -->
<div class="screen" id="screen-5">
  <div class="lbl">Umbrella Company</div>
  <h2 class="q-h">What is your umbrella company margin fee per week?</h2>
  <p class="q-s">The weekly management fee charged by your umbrella company.</p>
  <div class="inp-row">
    <span class="inp-pfx">£</span>
    <input class="num" id="umbrella-fee" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="20" autocomplete="off"
      value="20" oninput="syncUI()">
  </div>
  <div class="inp-hint">Most umbrella companies charge £15–£30 per week</div>
</div>
```

Then insert screens 7–8 between `screen-6` (pension yes/no) and `screen-9` (output):

```html
<!-- SCREEN 7 — Pension percentage -->
<div class="screen" id="screen-7">
  <div class="lbl">Pension</div>
  <h2 class="q-h">What percentage of gross do you contribute?</h2>
  <p class="q-s">Your contribution is taken from your employee gross pay.</p>
  <div class="inp-row">
    <input class="num" id="pension-pct" type="number" inputmode="numeric"
      pattern="[0-9]*" placeholder="0" autocomplete="off" oninput="syncUI()">
    <span class="inp-pfx" style="padding-left:4px;padding-right:0">%</span>
  </div>
  <div class="inp-hint">Auto-enrolment minimum is 5% employee contribution</div>
</div>

<!-- SCREEN 8 — Employed elsewhere -->
<div class="screen" id="screen-8">
  <div class="lbl">Other Income</div>
  <h2 class="q-h">Are you employed elsewhere?</h2>
  <p class="q-s">This can affect your personal allowance and NI thresholds.</p>
  <div class="cards">

    <div class="card" data-group="employedElsewhere" data-value="no" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No</div>
        <div class="c-desc">This is my only source of income</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="employedElsewhere" data-value="yes" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Yes</div>
        <div class="c-desc">I also have a PAYE job or other employment</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

  </div>
  <div class="emp-warning" id="employed-warning">
    <p>&#9888; Your personal allowance and NI thresholds may already be used by your employment.
    The figures shown may not reflect your actual tax liability — please speak to an accountant.</p>
  </div>
</div>
```

- [ ] **Step 5: Open in browser. Count 10 `.screen` divs in DevTools (Elements panel). Verify the DOM order is 0,1,2,3,4,5,6,7,8,9 with the correct IDs. Console should show no errors.**

---

## Task 3: End-to-end verification

- [ ] **Step 1: Test the full Inside IR35 happy path**

Walk through: Welcome → Inside IR35 → Day Rate (£500) → 5 days/week → 20 days holiday → £20 umbrella fee → Pension: No → Employed elsewhere: No → Output screen.

Expected on output screen:
- Working days = (52 × 5) − 20 = 240
- Annual gross = £500 × 240 = £120,000
- Employer NI = (£120,000 − £5,000) × 15% = £17,250
- Levy = £120,000 × 0.5% = £600
- Umbrella annual = £20 × 52 = £1,040
- Employee gross = £120,000 − £1,040 − £17,250 − £600 = £101,110
- Personal allowance tapers: PA = max(0, 12570 − floor((101110−100000)/2)) = max(0, 12570 − 555) = £12,015
- Taxable = £101,110 − £12,015 = £89,095
  - Basic band: min(89095, 37700) × 20% = £7,540
  - Higher band: min(89095−37700, 74870) × 40% = £51,395 × 40% = £20,558
  - Income tax ≈ £28,098
- Employee NI:
  - £12,570–£50,270: £37,700 × 8% = £3,016
  - £50,270–£101,110: £50,840 × 2% = £1,016.80
  - Total ≈ £4,032.80
- Net = £101,110 − £28,098 − £4,033 ≈ £68,979

Verify the number shown on screen roughly matches (within £100 rounding tolerance).

- [ ] **Step 2: Test the pension branch**

Redo with Pension: Yes → 5% → verify screen-7 (pension %) appears and output shows pension deduction.

- [ ] **Step 3: Test the employed elsewhere warning**

On screen-8, tap "Yes". Verify warning banner appears and Continue button remains (is NOT auto-hidden or disabled). Tap Continue — verify it advances to output.

- [ ] **Step 4: Test Back navigation**

On the output screen, tap Back. Verify it returns to screen-8 (employed elsewhere), not screen-9 or screen-7.

On screen-8, tap Back. If pension was "Yes", verify it returns to screen-7 (pension %); if "No", verify it returns to screen-6 (pension yes/no).

- [ ] **Step 5: Test Recalculate**

On output screen, tap Recalculate. Verify:
- Returns to Welcome (screen-0)
- All inputs are cleared
- Umbrella fee resets to £20
- All card selections are cleared
- Progress bar resets to 0%

- [ ] **Step 6: Test count-up and stagger animations**

On the output screen, verify:
- All £ values count up from £0 over ~800ms on first arrival
- Output cards stagger in with visible delay between each
- Pension row is hidden when pension=No, visible when pension=Yes

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add Inside IR35 question flow and calculated output"
```

---

## Self-review notes

**Spec coverage check:**
- Day rate screen ✓ (existing screen-2)
- Days per week (cards 1–5) ✓ (screen-3)
- Days holiday (number input) ✓ (screen-4)
- Umbrella fee with £20 default ✓ (screen-5)
- Pension yes/no ✓ (screen-6)
- Pension % conditional ✓ (screen-7, gated by `getNextScreen`)
- Employed elsewhere + warning banner ✓ (screen-8)
- Calculations: all 2026/27 rates applied ✓ (calculateIR35)
- Output: all items listed in spec ✓ (screen-9)
- Count-up 800ms ✓ (DUR=800 in countUp)
- Stagger 100ms ✓ (staggerIn with i*100 delay)
- Disclaimer ✓ (bottom of screen-9)
- Employer NI highlighted ✓ (ni-highlight class + ni-badge)
- Effective tax rate ✓ (o-eff-rate, set in renderOutput not counted up)

**Notes:**
- The `o-eff-rate` element shows a percentage, not a £ value — it is populated by `renderOutput()` and not included in the count-up items, which is correct.
- `el.closest` is used in `countUp` — available in all modern browsers including iPhone Safari 9+. Safe.
- `umbrella-fee` is pre-filled with `value="20"` in HTML so screen-5 ok() returns true on entry.
