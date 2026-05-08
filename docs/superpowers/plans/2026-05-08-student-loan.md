# Student Loan Repayment Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Do you have a student loan?" question (6-card 2-column layout) to all journeys — Inside IR35, Ltd Co single director, and Ltd Co director loop — with per-plan repayment calculations reflected in all output screens.

**Architecture:** All changes are in the single `index.html` file. New screens 26 (IR35), 27 (Ltd Co single dir), and 28 (director loop) are appended at the end of the DOM; `getNextScreen` routes to them. A `STUDENT_LOAN_PLANS` lookup table and `calcStudentLoan` helper keep calculation logic in one place. Student loan is stored in `ans.studentLoan` for IR35/single-director, and per-director in `ans.directors[n].studentLoan` for the loop. This plan also fixes a pre-existing routing bug: screen 17 (Ltd Co pension question) is currently unreachable for single directors.

**Tech Stack:** Vanilla JS, HTML, CSS — no libraries.

---

## Critical context before touching anything

- `screens` array is built from `document.querySelectorAll('.screen')` in DOM order — screen IDs match their index. New screens appended after `#screen-25` become indices 26, 27, 28.
- `getNextScreen(idx)` drives all navigation. `history` is a push/pop stack. `goTo(idx, dir)` handles the animation.
- `ans` object holds all answers. `ans.directors[n]` holds per-director state populated by `pickCard` and `saveCurrentDirector`.
- **Bug already in the code:** `getNextScreen(16)` for single director goes to `ltdPension=yes?18:19`, skipping screen 17 (the pension yes/no question). Fix this as part of Task 6.
- `loopPct(fieldIdx)` divides progress across `fields*directors`. Currently 6 fields. Adding screen 28 makes it 7 fields — update the constant in the formula.
- `populateLoopScreens()` is called only from `onEnter(20)` — it resets/restores all loop screen state. Update it to cover screen 28.
- `autoAdvance` in `pickCard` fires after 390ms when ok(). Student loan cards auto-advance (they are not in the `isEmpElsewhere` check).

---

## File

- Modify: `index.html` (all changes are here)
- Modify: `CLAUDE.md` (screen map, navigation, build status)

---

## Task 1: Add STUDENT_LOAN_PLANS constant and calcStudentLoan helper

**Files:** Modify `index.html`

- [ ] **Step 1: Add constant and helper after EXP_CATS (line ~1021)**

Find this block in `index.html`:
```javascript
var EXP_CATS = [
  ...
];

function getExpTotal() {
```

Insert after the closing `];` of EXP_CATS and before `function getExpTotal()`:

```javascript
var STUDENT_LOAN_PLANS = {
  none:    { threshold: 0,     rate: 0    },
  plan1:   { threshold: 24990, rate: 0.09 },
  plan2:   { threshold: 27295, rate: 0.09 },
  plan4:   { threshold: 31395, rate: 0.09 },
  plan5:   { threshold: 25000, rate: 0.09 },
  postgrad:{ threshold: 21000, rate: 0.06 }
};

function calcStudentLoan(plan, income) {
  var p = STUDENT_LOAN_PLANS[plan] || STUDENT_LOAN_PLANS.none;
  if (!p.rate) return 0;
  return Math.max(0, (income - p.threshold) * p.rate);
}

```

- [ ] **Step 2: Update `ans` initialisation to include `studentLoan`**

Find:
```javascript
var ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
            ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
            directors: [] };
```

Replace with:
```javascript
var ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
            ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
            studentLoan: null, directors: [] };
```

- [ ] **Step 3: Update `initDirLoop` to include `studentLoan: null` per director**

Find inside `function initDirLoop()`:
```javascript
    ans.directors.push({
      name: 'Director ' + (i + 1),
      shareholding: 0,
      salary: 12570,
      employedElsewhere: null,
      pension: null,
      pensionMonthly: 0
    });
```

Replace with:
```javascript
    ans.directors.push({
      name: 'Director ' + (i + 1),
      shareholding: 0,
      salary: 12570,
      employedElsewhere: null,
      pension: null,
      pensionMonthly: 0,
      studentLoan: null
    });
```

- [ ] **Step 4: Update `resetAll` to reset `studentLoan`**

Find inside `function resetAll()`:
```javascript
  ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
          ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
          directors: [] };
```

Replace with:
```javascript
  ans = { ir35: null, daysPerWeek: null, pension: null, employedElsewhere: null,
          ltdDaysPerWeek: null, numDirectors: null, ltdEmployedElsewhere: null, ltdPension: null,
          studentLoan: null, directors: [] };
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add STUDENT_LOAN_PLANS constant and calcStudentLoan helper, initialise studentLoan state"
```

---

## Task 2: Add CSS for 2-column student loan card grid

**Files:** Modify `index.html`

- [ ] **Step 1: Add `.cards-sl` and `.sl-full` CSS**

Find this comment in the CSS section:
```css
/* ── Expense category grid (screen 16) ── */
```

Insert the following block **before** that comment:

```css
/* ── Student loan 2-column card grid ── */
.cards-sl{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.cards-sl .card{min-height:72px;padding:14px 12px;align-items:flex-start}
.cards-sl .c-icon{display:none}
.cards-sl .card.selected .c-icon{display:none}
.sl-full{grid-column:span 2}

```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add CSS for 2-column student loan card grid"
```

---

## Task 3: Add 3 new screen divs and output rows

**Files:** Modify `index.html`

The SVG icons below are the same check/arrow SVGs used throughout the file. Copy them exactly.

**Check SVG:**
```html
<svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
```

**Arrow SVG:**
```html
<svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
```

### 3a — Screen 26 (Inside IR35 student loan)

- [ ] **Step 1: Insert screen-26 before the expenses total bar**

Find:
```html
<!-- EXPENSES TOTAL (fixed bar, shown only on screen 16) -->
```

Insert the following block **immediately before** that line:

```html
<!-- SCREEN 26 — Inside IR35: Student loan -->
<div class="screen" id="screen-26">
  <div class="lbl">Student Loan</div>
  <h2 class="q-h">Do you have a student loan?</h2>
  <p class="q-s">Repayments are deducted from your salary above the threshold.</p>
  <div class="cards cards-sl">

    <div class="card sl-full" data-group="studentLoan" data-value="none" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No student loan</div>
        <div class="c-desc">No repayments due</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan1" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 1</div>
        <div class="c-desc">£24,990 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan2" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 2</div>
        <div class="c-desc">£27,295 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan4" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 4</div>
        <div class="c-desc">£31,395 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan5" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 5</div>
        <div class="c-desc">£25,000 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card sl-full" data-group="studentLoan" data-value="postgrad" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Postgraduate Loan</div>
        <div class="c-desc">£21,000 · 6%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

  </div>
</div>

```

### 3b — Screen 27 (Ltd Co single director student loan)

- [ ] **Step 2: Insert screen-27 immediately after the screen-26 closing div**

Insert directly after the `</div>` that closes screen-26, before the expenses total comment:

```html
<!-- SCREEN 27 — Ltd Co single director: Student loan -->
<div class="screen" id="screen-27">
  <div class="lbl">Student Loan</div>
  <h2 class="q-h">Do you have a student loan?</h2>
  <p class="q-s">Repayments are deducted from your salary and dividends above the threshold.</p>
  <div class="cards cards-sl">

    <div class="card sl-full" data-group="studentLoan" data-value="none" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No student loan</div>
        <div class="c-desc">No repayments due</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan1" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 1</div>
        <div class="c-desc">£24,990 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan2" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 2</div>
        <div class="c-desc">£27,295 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan4" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 4</div>
        <div class="c-desc">£31,395 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="studentLoan" data-value="plan5" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 5</div>
        <div class="c-desc">£25,000 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card sl-full" data-group="studentLoan" data-value="postgrad" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Postgraduate Loan</div>
        <div class="c-desc">£21,000 · 6%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

  </div>
</div>

```

### 3c — Screen 28 (Director loop student loan)

- [ ] **Step 3: Insert screen-28 immediately after the screen-27 closing div**

```html
<!-- SCREEN 28 — Director loop: Student loan -->
<div class="screen" id="screen-28">
  <div class="lbl" id="dir-loop-lbl-28">Director 1 of 2</div>
  <h2 class="q-h">Does this director have a student loan?</h2>
  <p class="q-s">Repayments are deducted from their salary and dividends above the threshold.</p>
  <div class="cards cards-sl">

    <div class="card sl-full" data-group="dirStudentLoan" data-value="none" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">No student loan</div>
        <div class="c-desc">No repayments due</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="dirStudentLoan" data-value="plan1" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 1</div>
        <div class="c-desc">£24,990 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="dirStudentLoan" data-value="plan2" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 2</div>
        <div class="c-desc">£27,295 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="dirStudentLoan" data-value="plan4" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 4</div>
        <div class="c-desc">£31,395 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card" data-group="dirStudentLoan" data-value="plan5" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Plan 5</div>
        <div class="c-desc">£25,000 · 9%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

    <div class="card sl-full" data-group="dirStudentLoan" data-value="postgrad" onclick="pickCard(this)">
      <div class="card-body">
        <div class="c-title">Postgraduate Loan</div>
        <div class="c-desc">£21,000 · 6%</div>
      </div>
      <div class="c-icon">
        <svg class="ico-chk" width="22" height="22" viewBox="0 0 22 22" fill="none"><path d="M4 11L9 16L18 7" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <svg class="ico-arr" width="18" height="18" viewBox="0 0 18 18" fill="none"><path d="M6 9H12M12 9L9.5 6.5M12 9L9.5 11.5" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/></svg>
      </div>
    </div>

  </div>
</div>

```

### 3d — Student loan output row on screen 9

- [ ] **Step 4: Add student loan output row to screen 9 (Inside IR35 output)**

Find inside `#screen-9`:
```html
    <div class="out-card" id="o-pension-row">
      <div class="o-lbl">Pension Contribution</div>
      <div class="o-val neg" id="o-pension">-£0</div>
      <div class="o-sub" id="o-pension-sub"></div>
    </div>

    <div class="disclaimer">
```

Replace with:
```html
    <div class="out-card" id="o-pension-row">
      <div class="o-lbl">Pension Contribution</div>
      <div class="o-val neg" id="o-pension">-£0</div>
      <div class="o-sub" id="o-pension-sub"></div>
    </div>

    <div class="out-card" id="o-student-loan-row">
      <div class="o-lbl">Student Loan Repayment</div>
      <div class="o-val neg" id="o-student-loan">-£0</div>
      <div class="o-sub" id="o-student-loan-sub"></div>
    </div>

    <div class="disclaimer">
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add HTML for screens 26/27/28 (student loan) and output row on screen 9"
```

---

## Task 4: SCREENS config entries (26, 27, 28) and loopPct fix

**Files:** Modify `index.html`

- [ ] **Step 1: Update loopPct to use 7 fields**

Find:
```javascript
function loopPct(fieldIdx) {
  var n = dirLoop.current;
  var total = +ans.numDirectors || 1;
  return Math.round(36 + 54 * (n * 6 + fieldIdx) / (total * 6));
}
```

Replace with:
```javascript
function loopPct(fieldIdx) {
  var n = dirLoop.current;
  var total = +ans.numDirectors || 1;
  return Math.round(36 + 54 * (n * 7 + fieldIdx) / (total * 7));
}
```

- [ ] **Step 2: Update Inside IR35 step counts from "X of 7" to "X of 8"**

Find and replace these entries in the SCREENS object (screens 1–8):

```javascript
  1: { pct:12,  back:true,  step:'1 of 7', btn:'Continue',    ok:function(){ return ans.ir35 !== null; } },
  2: { pct:24,  back:true,  step:'2 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('day-rate').value; return v!==''&&+v>0; } },
  3: { pct:36,  back:true,  step:'3 of 7', btn:'Continue',    ok:function(){ return ans.daysPerWeek !== null; } },
  4: { pct:48,  back:true,  step:'4 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('holiday-days').value; return v!==''&&+v>=0&&+v<365; } },
  5: { pct:60,  back:true,  step:'5 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('umbrella-fee').value; return v!==''&&+v>=0; } },
  6: { pct:72,  back:true,  step:'6 of 7', btn:'Continue',    ok:function(){ return ans.pension !== null; } },
  7: { pct:80,  back:true,  step:'6 of 7', btn:'Continue',    ok:function(){ var v=document.getElementById('pension-pct').value; return v!==''&&+v>0&&+v<=100; } },
  8: { pct:88,  back:true,  step:'7 of 7', btn:'Continue',    ok:function(){ return ans.employedElsewhere !== null; } },
```

Replace with:
```javascript
  1: { pct:11,  back:true,  step:'1 of 8', btn:'Continue',    ok:function(){ return ans.ir35 !== null; } },
  2: { pct:22,  back:true,  step:'2 of 8', btn:'Continue',    ok:function(){ var v=document.getElementById('day-rate').value; return v!==''&&+v>0; } },
  3: { pct:33,  back:true,  step:'3 of 8', btn:'Continue',    ok:function(){ return ans.daysPerWeek !== null; } },
  4: { pct:44,  back:true,  step:'4 of 8', btn:'Continue',    ok:function(){ var v=document.getElementById('holiday-days').value; return v!==''&&+v>=0&&+v<365; } },
  5: { pct:55,  back:true,  step:'5 of 8', btn:'Continue',    ok:function(){ var v=document.getElementById('umbrella-fee').value; return v!==''&&+v>=0; } },
  6: { pct:66,  back:true,  step:'6 of 8', btn:'Continue',    ok:function(){ return ans.pension !== null; } },
  7: { pct:72,  back:true,  step:'6 of 8', btn:'Continue',    ok:function(){ var v=document.getElementById('pension-pct').value; return v!==''&&+v>0&&+v<=100; } },
  8: { pct:77,  back:true,  step:'7 of 8', btn:'Continue',    ok:function(){ return ans.employedElsewhere !== null; } },
```

- [ ] **Step 3: Update Ltd Co single-director step counts (screens 14–18) to reflect the routing fix (screen 17 now reachable) and new student loan screen (27)**

Find these entries:
```javascript
  14: { pct:45,  back:true, step:'5 of 9', btn:'Continue',    ok:function(){ var v=document.getElementById('ltd-salary').value; return v!==''&&+v>=0; } },
  15: { pct:54,  back:true, step:'6 of 9', btn:'Continue',    ok:function(){ return ans.ltdEmployedElsewhere !== null; } },
  16: { pct:function(){return +ans.numDirectors>1?92:63;}, back:true, step:'7 of 9', btn:'Continue', ok:function(){ return getExpTotal() >= 1; } },
  17: { pct:72,  back:true, step:'8 of 9', btn:'Continue',    ok:function(){ return ans.ltdPension !== null; } },
  18: { pct:82,  back:true, step:'8 of 9', btn:'Continue',    ok:function(){ var v=document.getElementById('ltd-pension-amount').value; return v!==''&&+v>0; } },
```

Replace with:
```javascript
  14: { pct:40,  back:true, step:'5 of 10', btn:'Continue',    ok:function(){ var v=document.getElementById('ltd-salary').value; return v!==''&&+v>=0; } },
  15: { pct:50,  back:true, step:'6 of 10', btn:'Continue',    ok:function(){ return ans.ltdEmployedElsewhere !== null; } },
  16: { pct:function(){return +ans.numDirectors>1?92:60;}, back:true, step:'7 of 10', btn:'Continue', ok:function(){ return getExpTotal() >= 1; } },
  17: { pct:70,  back:true, step:'8 of 10', btn:'Continue',    ok:function(){ return ans.ltdPension !== null; } },
  18: { pct:80,  back:true, step:'8 of 10', btn:'Continue',    ok:function(){ var v=document.getElementById('ltd-pension-amount').value; return v!==''&&+v>0; } },
```

- [ ] **Step 4: Add SCREENS entries for 26, 27, 28 after the closing of screen 25's entry**

Find:
```javascript
  25: { pct: function(){return loopPct(5);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var el=document.getElementById('dir-pension-amount-loop'); return !!(el&&el.value!==''&&+el.value>0); } },
};
```

Replace with:
```javascript
  25: { pct: function(){return loopPct(5);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var el=document.getElementById('dir-pension-amount-loop'); return !!(el&&el.value!==''&&+el.value>0); } },
  26: { pct:88,  back:true, step:'8 of 8',  btn:'Continue', ok:function(){ return ans.studentLoan !== null; } },
  27: { pct:90,  back:true, step:'9 of 10', btn:'Continue', ok:function(){ return ans.studentLoan !== null; } },
  28: { pct: function(){return loopPct(6);}, back:true, step: loopStep, btn:'Continue',
        ok: function(){ var d=ans.directors[dirLoop.current]; return !!(d&&d.studentLoan!==null&&d.studentLoan!==undefined); } },
};
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add SCREENS config for screens 26/27/28, update step counts, fix loopPct to 7 fields"
```

---

## Task 5: Navigation — getNextScreen and goBack

**Files:** Modify `index.html`

- [ ] **Step 1: Replace the entire `getNextScreen` function**

Find:
```javascript
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

Replace with:
```javascript
function getNextScreen(idx) {
  if (idx === 0) return 1;
  if (idx === 1) return ans.ir35 === 'outside' ? 10 : 2;
  if (idx === 2) return 3;
  if (idx === 3) return 4;
  if (idx === 4) return 5;
  if (idx === 5) return 6;
  if (idx === 6) return ans.pension === 'yes' ? 7 : 8;
  if (idx === 7) return 8;
  if (idx === 8) return 26;
  if (idx === 26) return 9;
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
  if (idx === 16) return +ans.numDirectors > 1 ? 19 : 17;
  if (idx === 17) return ans.ltdPension === 'yes' ? 18 : 27;
  if (idx === 18) return 27;
  if (idx === 27) return 19;
  if (idx === 20) return 21;
  if (idx === 21) return 22;
  if (idx === 22) return 23;
  if (idx === 23) return 24;
  if (idx === 24) {
    if (ans.directors[dirLoop.current] && ans.directors[dirLoop.current].pension === 'yes') return 25;
    return 28;
  }
  if (idx === 25) return 28;
  if (idx === 28) {
    saveCurrentDirector();
    return nextDirectorOrExit();
  }
  return null;
}
```

Note the key changes:
- `idx 8 → 26` (was 9) then `idx 26 → 9`
- `idx 16 → 17` for single director (was the broken `ltdPension` check — this fixes the routing bug)
- `idx 17 → 27` when no pension (was 19) and `idx 18 → 27` (was 19) and `idx 27 → 19`
- `idx 24 → 28` instead of calling `saveCurrentDirector` early; `idx 25 → 28`; `idx 28` does the save

- [ ] **Step 2: Update `goBack` to handle prev === 28 for cross-director boundary**

Find:
```javascript
window.goBack = function(){
  if (history.length === 0) return;
  var prev = history.pop();
  if (cur === 20 && (prev === 24 || prev === 25) && dirLoop.current > 0) {
    dirLoop.current--;
  }
  goTo(prev, -1);
};
```

Replace with:
```javascript
window.goBack = function(){
  if (history.length === 0) return;
  var prev = history.pop();
  if (cur === 20 && (prev === 24 || prev === 25 || prev === 28) && dirLoop.current > 0) {
    dirLoop.current--;
  }
  goTo(prev, -1);
};
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: update navigation routing for student loan screens, fix screen 17 unreachable bug"
```

---

## Task 6: pickCard handler, populateLoopScreens, and onEnter

**Files:** Modify `index.html`

- [ ] **Step 1: Add dirStudentLoan handler in pickCard**

Find inside `window.pickCard`:
```javascript
  if (grp === 'dirEmployedElsewhere') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].employedElsewhere = el.dataset.value;
  } else if (grp === 'dirPension') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].pension = el.dataset.value;
  } else {
    ans[grp] = el.dataset.value;
  }
```

Replace with:
```javascript
  if (grp === 'dirEmployedElsewhere') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].employedElsewhere = el.dataset.value;
  } else if (grp === 'dirPension') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].pension = el.dataset.value;
  } else if (grp === 'dirStudentLoan') {
    if (ans.directors[dirLoop.current]) ans.directors[dirLoop.current].studentLoan = el.dataset.value;
  } else {
    ans[grp] = el.dataset.value;
  }
```

- [ ] **Step 2: Update populateLoopScreens to include screen 28 label and restore student loan card state**

Find:
```javascript
function populateLoopScreens() {
  var n = dirLoop.current;
  var total = +ans.numDirectors;
  var label = 'Director ' + (n + 1) + ' of ' + total;
  [20, 21, 22, 23, 24, 25].forEach(function(i) {
    var lbl = document.getElementById('dir-loop-lbl-' + i);
    if (lbl) lbl.textContent = label;
  });
```

Replace with:
```javascript
function populateLoopScreens() {
  var n = dirLoop.current;
  var total = +ans.numDirectors;
  var label = 'Director ' + (n + 1) + ' of ' + total;
  [20, 21, 22, 23, 24, 25, 28].forEach(function(i) {
    var lbl = document.getElementById('dir-loop-lbl-' + i);
    if (lbl) lbl.textContent = label;
  });
```

Then find the block that restores pension card selection (near the end of populateLoopScreens):
```javascript
  if (d.pension) {
    var pCard = document.querySelector('[data-group="dirPension"][data-value="' + d.pension + '"]');
    if (pCard) pCard.classList.add('selected');
  }
  syncShareholding();
}
```

Replace with:
```javascript
  if (d.pension) {
    var pCard = document.querySelector('[data-group="dirPension"][data-value="' + d.pension + '"]');
    if (pCard) pCard.classList.add('selected');
  }
  [].forEach.call(document.querySelectorAll('[data-group="dirStudentLoan"]'), function(c){
    c.classList.remove('selected');
  });
  if (d.studentLoan) {
    var slCard = document.querySelector('[data-group="dirStudentLoan"][data-value="' + d.studentLoan + '"]');
    if (slCard) slCard.classList.add('selected');
  }
  syncShareholding();
}
```

- [ ] **Step 3: Add onEnter handlers for screens 26, 27, 28**

Find:
```javascript
  if (idx === 25) setTimeout(function(){ var el=document.getElementById('dir-pension-amount-loop'); if(el) el.focus(); }, 340);
  if (idx === 19) {
```

Insert before the `if (idx === 19)` line:

```javascript
  if (idx === 26) {
    [].forEach.call(document.querySelectorAll('[data-group="studentLoan"]'), function(c){
      c.classList.remove('selected');
    });
    if (ans.studentLoan) {
      var slEl = document.querySelector('#screen-26 [data-group="studentLoan"][data-value="' + ans.studentLoan + '"]');
      if (slEl) slEl.classList.add('selected');
    }
  }
  if (idx === 27) {
    [].forEach.call(document.querySelectorAll('[data-group="studentLoan"]'), function(c){
      c.classList.remove('selected');
    });
    if (ans.studentLoan) {
      var slEl27 = document.querySelector('#screen-27 [data-group="studentLoan"][data-value="' + ans.studentLoan + '"]');
      if (slEl27) slEl27.classList.add('selected');
    }
  }
  if (idx === 28) {
    [].forEach.call(document.querySelectorAll('[data-group="dirStudentLoan"]'), function(c){
      c.classList.remove('selected');
    });
    var dsl = ans.directors[dirLoop.current];
    if (dsl && dsl.studentLoan) {
      var slEl28 = document.querySelector('[data-group="dirStudentLoan"][data-value="' + dsl.studentLoan + '"]');
      if (slEl28) slEl28.classList.add('selected');
    }
  }
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: wire up pickCard for dirStudentLoan, restore state in populateLoopScreens and onEnter"
```

---

## Task 7: calculateIR35 and renderOutput updates

**Files:** Modify `index.html`

- [ ] **Step 1: Add studentLoanRepayment to calculateIR35**

Find inside `function calculateIR35()`:
```javascript
  var pension    = employeeGross * (pensionPct / 100);
  var netAnnual  = employeeGross - incomeTax - employeeNI - pension;
  var netMonthly = netAnnual / 12;
  var totalDeds  = umbrellaAnnual + employerNI + levy + incomeTax + employeeNI + pension;
  // Spec: "Effective rate = Net ÷ Gross Contract Value × 100"
  var effRate    = annualGross > 0 ? (netAnnual / annualGross) * 100 : 0;

  return {
    annualGross:annualGross, workingDays:workingDays, umbrellaAnnual:umbrellaAnnual,
    employerNI:employerNI, levy:levy, employeeGross:employeeGross,
    incomeTax:incomeTax, employeeNI:employeeNI, pension:pension,
    netAnnual:netAnnual, netMonthly:netMonthly, effRate:effRate
  };
```

Replace with:
```javascript
  var pension              = employeeGross * (pensionPct / 100);
  var studentLoanRepayment = calcStudentLoan(ans.studentLoan || 'none', employeeGross);
  var netAnnual            = employeeGross - incomeTax - employeeNI - pension - studentLoanRepayment;
  var netMonthly           = netAnnual / 12;
  // Spec: "Effective rate = Net ÷ Gross Contract Value × 100"
  var effRate              = annualGross > 0 ? (netAnnual / annualGross) * 100 : 0;

  return {
    annualGross:annualGross, workingDays:workingDays, umbrellaAnnual:umbrellaAnnual,
    employerNI:employerNI, levy:levy, employeeGross:employeeGross,
    incomeTax:incomeTax, employeeNI:employeeNI, pension:pension,
    studentLoanRepayment:studentLoanRepayment,
    netAnnual:netAnnual, netMonthly:netMonthly, effRate:effRate
  };
```

- [ ] **Step 2: Update renderOutput to show/hide the student loan row**

Find inside `function renderOutput()`:
```javascript
  var effEl = document.getElementById('o-eff-rate');
  if (effEl) effEl.textContent = r.effRate.toFixed(1) + '%';
}
```

Replace with:
```javascript
  var slRow = document.getElementById('o-student-loan-row');
  if (slRow) {
    var showSl = r.studentLoanRepayment > 0;
    slRow.style.display = showSl ? '' : 'none';
    if (showSl) {
      var planLabels = { plan1:'Plan 1', plan2:'Plan 2', plan4:'Plan 4', plan5:'Plan 5', postgrad:'Postgraduate Loan' };
      var slSub = document.getElementById('o-student-loan-sub');
      if (slSub) slSub.textContent = (planLabels[ans.studentLoan] || '') + ' repayment';
    }
  }
  var effEl = document.getElementById('o-eff-rate');
  if (effEl) effEl.textContent = r.effRate.toFixed(1) + '%';
}
```

- [ ] **Step 3: Add student loan to resetNums**

Find:
```javascript
function resetNums() {
  ['o-gross','o-umbrella','o-employer-ni','o-levy','o-employee-ni',
   'o-income-tax','o-pension','o-net-annual','o-net-monthly'].forEach(function(id){
    var el = document.getElementById(id);
    if (el) el.textContent = '£0';
  });
}
```

Replace with:
```javascript
function resetNums() {
  ['o-gross','o-umbrella','o-employer-ni','o-levy','o-employee-ni',
   'o-income-tax','o-pension','o-student-loan','o-net-annual','o-net-monthly'].forEach(function(id){
    var el = document.getElementById(id);
    if (el) el.textContent = '£0';
  });
}
```

- [ ] **Step 4: Add student loan to countUp ITEMS for IR35**

Find inside `countUp()` (the else branch for IR35, where `ITEMS` is built for screen 9):
```javascript
    ITEMS = [
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
```

Replace with:
```javascript
    ITEMS = [
      { id:'o-gross',        val:r.annualGross,           neg:false },
      { id:'o-umbrella',     val:r.umbrellaAnnual,        neg:true  },
      { id:'o-employer-ni',  val:r.employerNI,            neg:true  },
      { id:'o-levy',         val:r.levy,                  neg:true  },
      { id:'o-employee-ni',  val:r.employeeNI,            neg:true  },
      { id:'o-income-tax',   val:r.incomeTax,             neg:true  },
      { id:'o-pension',      val:r.pension,               neg:true  },
      { id:'o-student-loan', val:r.studentLoanRepayment,  neg:true  },
      { id:'o-net-annual',   val:r.netAnnual,             neg:false },
      { id:'o-net-monthly',  val:r.netMonthly,            neg:false },
    ];
```

- [ ] **Step 5: Add student loan skip logic inside the countUp frame function**

The countUp loop skips hidden rows. Find inside the `frame` function inside `countUp`:
```javascript
      var ltdPRow = document.getElementById('o-ltd-pension-row');
      if (item.id === 'o-ltd-pension-corp' && ltdPRow && ltdPRow.style.display === 'none') return;
      var eniRow = document.getElementById('o-ltd-employer-ni-row');
      if (item.id === 'o-ltd-employer-ni' && eniRow && eniRow.style.display === 'none') return;
```

Add after those lines:
```javascript
      var slRow = document.getElementById('o-student-loan-row');
      if (item.id === 'o-student-loan' && slRow && slRow.style.display === 'none') return;
```

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add student loan calculation and output to Inside IR35 journey"
```

---

## Task 8: calculateLtdCo and renderLtdOutput updates

**Files:** Modify `index.html`

- [ ] **Step 1: Pass studentLoan into director objects inside calculateLtdCo**

Find inside `calculateLtdCo()` where `directors` is built for single director:
```javascript
    directors = [{
      name:              'Director',
      shareholding:      100,
      salary:            +document.getElementById('ltd-salary').value || 0,
      employedElsewhere: ans.ltdEmployedElsewhere || 'no',
      pension:           ans.ltdPension || 'no',
      pensionMonthly:    ans.ltdPension === 'yes' ? (+document.getElementById('ltd-pension-amount').value || 0) : 0
    }];
```

Replace with:
```javascript
    directors = [{
      name:              'Director',
      shareholding:      100,
      salary:            +document.getElementById('ltd-salary').value || 0,
      employedElsewhere: ans.ltdEmployedElsewhere || 'no',
      pension:           ans.ltdPension || 'no',
      pensionMonthly:    ans.ltdPension === 'yes' ? (+document.getElementById('ltd-pension-amount').value || 0) : 0,
      studentLoan:       ans.studentLoan || 'none'
    }];
```

Find where `directors` is built for multi-director:
```javascript
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
```

Replace with:
```javascript
    directors = ans.directors.map(function(d) {
      return {
        name:              d.name,
        shareholding:      +d.shareholding || 0,
        salary:            +d.salary || 0,
        employedElsewhere: d.employedElsewhere || 'no',
        pension:           d.pension || 'no',
        pensionMonthly:    d.pension === 'yes' ? (+d.pensionMonthly || 0) : 0,
        studentLoan:       d.studentLoan || 'none'
      };
    });
```

- [ ] **Step 2: Compute studentLoanRepayment per director in the dirResults mapping**

Find:
```javascript
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
```

Replace with:
```javascript
    var studentLoanRepayment = calcStudentLoan(d.studentLoan || 'none', d.salary + divReceived);
    return {
      name:                  d.name,
      shareholding:          d.shareholding,
      salary:                d.salary,
      studentLoan:           d.studentLoan || 'none',
      employedElsewhere:     d.employedElsewhere,
      dividendsReceived:     divReceived,
      incomeTax:             salTax,
      employeeNI:            employeeNI,
      netSalary:             netSalary,
      divTax:                divTax,
      studentLoanRepayment:  studentLoanRepayment,
      netTakeHome:           netSalary + divReceived - divTax - studentLoanRepayment
    };
```

- [ ] **Step 3: Update renderLtdOutput to add student loan rows inside director cards**

Find inside `renderLtdOutput()` the template string for director cards:
```javascript
        '<div class="out-row out-row-total"><span class="out-row-lbl out-row-lbl-bold">Director Net Take-Home</span>' +
          '<span class="out-row-val pos lg" id="o-dir-' + i + '-net">\xa30</span></div>' +
        warning +
        '</div>';
```

Replace with:
```javascript
        '<div class="out-row out-row-total"><span class="out-row-lbl out-row-lbl-bold">Director Net Take-Home</span>' +
          '<span class="out-row-val pos lg" id="o-dir-' + i + '-net">\xa30</span></div>' +
        (d.studentLoanRepayment > 0
          ? '<div class="out-row"><span class="out-row-lbl">Student Loan Repayment</span>' +
            '<span class="out-row-val neg" id="o-dir-' + i + '-student-loan">-\xa30</span></div>'
          : '') +
        warning +
        '</div>';
```

- [ ] **Step 4: Add student loan items to countUp for the Ltd Co branch**

Find inside `countUp()` where director items are pushed:
```javascript
      lr.directors.forEach(function(d, i) {
        ITEMS.push({ id:'o-dir-'+i+'-gross-salary', val:d.salary,            neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-net-salary',   val:d.netSalary,         neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-divs-received',val:d.dividendsReceived, neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-div-tax',      val:d.divTax,            neg:true  });
        ITEMS.push({ id:'o-dir-'+i+'-net',          val:d.netTakeHome,       neg:false });
      });
```

Replace with:
```javascript
      lr.directors.forEach(function(d, i) {
        ITEMS.push({ id:'o-dir-'+i+'-gross-salary', val:d.salary,                  neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-net-salary',   val:d.netSalary,               neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-divs-received',val:d.dividendsReceived,       neg:false });
        ITEMS.push({ id:'o-dir-'+i+'-div-tax',      val:d.divTax,                  neg:true  });
        ITEMS.push({ id:'o-dir-'+i+'-net',          val:d.netTakeHome,             neg:false });
        if (d.studentLoanRepayment > 0) {
          ITEMS.push({ id:'o-dir-'+i+'-student-loan', val:d.studentLoanRepayment,  neg:true  });
        }
      });
```

- [ ] **Step 5: Update resetLtdNums — student loan amounts are in dynamic director cards and get reset by renderLtdOutput rebuilding innerHTML, so no change needed here. Verify by confirming `resetLtdNums` only resets static IDs that persist across renders.**

The current `resetLtdNums` only resets static IDs (`o-ltd-net-annual`, `o-ltd-gross`, etc.). Director card elements are regenerated by `renderLtdOutput` each time, so they start at £0 automatically. No change needed.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add student loan calculation and output to Ltd Co journey (single director and loop)"
```

---

## Task 9: Update CLAUDE.md

**Files:** Modify `CLAUDE.md`

- [ ] **Step 1: Update the Inside IR35 screen map (screens 0–9)**

Change the screen map table to add screen 26 between screens 8 and 9:

```markdown
| 8  | screen-8  | Employed elsewhere? Yes/No + warning banner |
| 26 | screen-26 | Student loan? (6-card 2-col grid) |
| 9  | screen-9  | IR35 output — full calculated breakdown |
```

- [ ] **Step 2: Update the Ltd Co single-director screen map**

Update to show screen 17 is now reachable, and add screen 27:

```markdown
| 14 | screen-14 | Director salary (£ input, id `ltd-salary`, default £12,570) |
| 15 | screen-15 | Employed elsewhere? Yes/No + warning banner (data-group `ltdEmployedElsewhere`) |
| 16 | screen-16 | Monthly company expenses (categorised, shared) |
| 17 | screen-17 | Company pension? Yes/No cards (data-group `ltdPension`) |
| 18 | screen-18 | Monthly pension amount (£ input, id `ltd-pension-amount`, conditional) |
| 27 | screen-27 | Student loan? (6-card 2-col grid) |
```

- [ ] **Step 3: Update the director loop screen map (screens 20–25 → 20–28)**

Add screen 28 to the loop:

```markdown
| 28 | screen-28 | Student loan? (6-card 2-col grid; data-group `dirStudentLoan`) |
```

- [ ] **Step 4: Update the navigation architecture section**

Update the `getNextScreen` bullet points to reflect:
- `Screen 8 → screen 26; screen 26 → screen 9`
- `Screen 16 → screen 17 for single director (was broken, now fixed)`
- `Screen 17 → 18 if ltdPension=yes, else 27; screen 18 → 27; screen 27 → 19`
- `Screen 24 → 28 if pension=no; screen 25 → 28; screen 28 → saveCurrentDirector() then nextDirectorOrExit()`
- `goBack() also decrements dirLoop.current when prev === 28`
- `loopPct` now uses 7 fields (added student loan field index 6)

- [ ] **Step 5: Update STUDENT_LOAN_PLANS in a new "Student loan rates" section under the tax rates section**

Add after the existing tax rates block:

```markdown
## Student loan repayment rates — 2026/27
- Plan 1: 9% above £24,990
- Plan 2: 9% above £27,295
- Plan 4: 9% above £31,395
- Plan 5: 9% above £25,000
- Postgraduate Loan: 6% above £21,000
- Income base: grossSalary (IR35); salary + dividendsReceived per director (Ltd Co)
```

- [ ] **Step 6: Commit everything**

```bash
git add index.html CLAUDE.md
git commit -m "docs: update CLAUDE.md with student loan screens, routing fix, and repayment rates"
```

---

## Self-review against spec

| Spec requirement | Task |
|---|---|
| Question in IR35 journey | Task 3 (screen 26 HTML) + Task 5 (routing 8→26→9) |
| Question in Ltd Co single director | Task 3 (screen 27 HTML) + Task 5 (routing 17→27→19) |
| Question per director in loop | Task 3 (screen 28 HTML) + Task 5 (24→28, 25→28, 28→save) |
| 6 plan options (No/P1/P2/P4/P5/Postgrad) | Task 3 (all 3 screens) |
| 2-column card grid | Task 2 (CSS) + Task 3 (HTML classes) |
| UK 2026/27 thresholds and rates | Task 1 (STUDENT_LOAN_PLANS) |
| calcStudentLoan helper | Task 1 |
| IR35: income = grossSalary | Task 7 |
| Ltd Co: income = salary + dividendsReceived | Task 8 |
| Student loan deducted from IR35 net | Task 7, Step 1 |
| Student loan deducted from director netTakeHome | Task 8, Step 2 |
| Output row on screen 9 (hidden if £0) | Task 3 (HTML) + Task 7 (show/hide logic) |
| Student loan row per director card on screen 19 | Task 8, Step 3 |
| Count-up animation includes student loan | Task 7, Steps 4–5; Task 8, Step 4 |
| Fix routing bug: screen 17 unreachable for single dir | Task 5, Step 1 |
| goBack cross-director boundary handles screen 28 | Task 5, Step 2 |
| State restored on back navigation | Task 6, Step 3 (onEnter) |
| loopPct updated for 7 fields | Task 4, Step 1 |
| CLAUDE.md updated | Task 9 |

No gaps found.
