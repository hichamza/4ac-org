# Practical Hero Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the `index.html` hero so a first-time visitor instantly sees what the tool produces and what to do, by moving the existing input bar and method tabs into the hero as two labeled steps.

**Architecture:** Pure DOM-relocation + CSS. The existing `.input-bar` and `#methodTabs` nodes are physically moved into `<header>` with their children, IDs, classes, and ARIA intact, so all JavaScript (`document.querySelectorAll('.method-tab')`, `document.querySelector('.input-bar .presets')`, every `#colorInput`/`#hexInput` listener) keeps working untouched. No JS changes. No new dependencies. Single file.

**Tech Stack:** Single static `index.html` (HTML + inline CSS + inline JS). No build, no test framework. Verification = `grep` invariant checks + a manual browser checklist served via `python3 -m http.server 8080`.

---

## File Structure

Only one file changes: `c:\Users\hicha\.vscode\4ac-org\index.html`.

Regions touched (line numbers are pre-change references — re-locate by string match before editing, since earlier edits shift later lines):

- **CSS** `:53–64` (HERO block), `:316–318` (`.top-row-left .input-bar`), `:704–753` (`.method-tabs` / `.method-tab`), `:830–833` (responsive header).
- **Markup** `:986–1021` (`<header>` … wordmark, eyebrow, h1, lede, `#methodTabs`), `:1051–1102` (`.top-row` → `.top-row-left` → `.input-bar`).

Out of scope (do not touch): weights/method panels, color wheel, toolbar, palette output, `palette-algorithm.html`, `README.md`.

---

### Task 1: Copy — acronym expansion + version

Make `4ac = 4 Awesome Colorizers` explicit and bump the visible version. "Amazing" appears in `index.html` only at the wordmark `title` and the eyebrow.

**Files:**
- Modify: `index.html` (wordmark `<a>` line, eyebrow `<div>` line — currently ~989, ~992)

- [ ] **Step 1: Verify the exact current strings**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -n "Amazing" index.html
```
Expected output (2 lines):
```
989:          <a href="https://4ac.org" class="wordmark" title="4 Amazing Colorizers">
992:          <div class="eyebrow">4 Amazing Colorizers · Four methods · v0.3.5</div>
```

- [ ] **Step 2: Update the wordmark title attribute**

Replace:
```html
          <a href="https://4ac.org" class="wordmark" title="4 Amazing Colorizers">
```
With:
```html
          <a href="https://4ac.org" class="wordmark" title="4 Awesome Colorizers">
```

- [ ] **Step 3: Update the eyebrow text + version**

Replace:
```html
          <div class="eyebrow">4 Amazing Colorizers · Four methods · v0.3.5</div>
```
With:
```html
          <div class="eyebrow">4ac = 4 Awesome Colorizers · Four methods · v0.3.7</div>
```

- [ ] **Step 4: Verify no "Amazing" remains in index.html**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -c "Amazing" index.html; grep -n "Awesome" index.html
```
Expected: count line prints `0`; then 2 lines containing `4 Awesome Colorizers` (title attr) and `4ac = 4 Awesome Colorizers · Four methods · v0.3.7` (eyebrow).

- [ ] **Step 5: Commit**

```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: expand acronym to '4 Awesome Colorizers', bump visible version to v0.3.7"
```

---

### Task 2: Copy — outcome headline + sub

Replace the marketing headline/lede with plain outcome language.

**Files:**
- Modify: `index.html` (`<h1>` and `<p class="lede">` — currently ~993–994)

- [ ] **Step 1: Verify current strings**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -n 'Modern color pairing\|class="lede">Drop in a single hex' index.html
```
Expected: 2 matching lines (the `<h1>` and the `.lede`).

- [ ] **Step 2: Replace the headline**

Replace:
```html
          <h1>Modern color pairing,<br>your <em>method</em> of choice.</h1>
```
With:
```html
          <h1>One hex in. A full, accessible <em>palette</em> out.</h1>
```

- [ ] **Step 3: Replace the lede**

Replace:
```html
          <p class="lede">Drop in a single hex. Choose how the accents are picked: a weighted OKLCH scorer, classical HSL color theory, a curated material library, or a hybrid that falls back when math gets ugly.</p>
```
With:
```html
          <p class="lede">Primary + 3 accents + status colors + 100–900 ramps. Export-ready. No signup.</p>
```

- [ ] **Step 4: Verify**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -n 'One hex in. A full, accessible' index.html && grep -c "Modern color pairing" index.html
```
Expected: the new `<h1>` line prints; `Modern color pairing` count prints `0`.

- [ ] **Step 5: Commit**

```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: replace marketing headline with plain outcome copy"
```

---

### Task 3: Remove `.input-bar` from `.top-row`

Detach the input bar node from `.top-row-left` so Task 4 can re-insert it into the hero. The node and all its children are removed *as one block* — nothing inside it changes.

**Files:**
- Modify: `index.html` (`.top-row` → `.top-row-left` region, currently ~1051–1102)

- [ ] **Step 1: Confirm the block boundaries**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -n '<div class="top-row">\|<div class="top-row-left">\|<div class="input-bar">\|<div class="weights" data-m1-only>' index.html
```
Expected: 4 lines. `top-row` then `top-row-left` then `input-bar` are consecutive; `weights` is the element immediately after the input-bar's closing `</div>`.

- [ ] **Step 2: Delete the entire `.input-bar` block**

Delete the contiguous block that starts at the line `    <div class="input-bar">` and ends at the `    </div>` immediately preceding `    <div class="weights" data-m1-only>`. The result must change this:

```html
    <div class="top-row">
    <div class="top-row-left">
    <div class="input-bar">
      <!-- ...swatch-input, hex-field, OR divider, tag-zone, presets... -->
    </div>

    <div class="weights" data-m1-only>
```

into this (input-bar gone; `top-row-left` now opens directly onto `weights`):

```html
    <div class="top-row">
    <div class="top-row-left">

    <div class="weights" data-m1-only>
```

Do not alter `weights` or anything after it.

- [ ] **Step 3: Verify the input-bar is detached and the JS host selector still has a target later**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -c '<div class="input-bar">' index.html
```
Expected: `0` (it is currently removed; Task 4 re-adds exactly one). The page is intentionally broken between Task 3 and Task 4 — do not browser-test here; proceed straight to Task 4.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: detach .input-bar from .top-row (relocates in next commit)"
```

---

### Task 4: Rebuild `<header>` — single column, two steps, moved nodes

Replace the entire `<header>…</header>` block with a single-column hero that contains: wordmark, eyebrow, h1, lede, **Step 1** label + the relocated `.input-bar`, **Step 2** label + the relocated `#methodTabs` (with its old `.method-tabs-header` strip removed, since the Step 2 label replaces it).

**Files:**
- Modify: `index.html` (`<header>…</header>`, currently ~986–1021)

- [ ] **Step 1: Confirm the current header span**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && grep -n '<header>\|</header>\|<div class="header-row">\|<div class="header-text">\|class="method-tabs"' index.html
```
Expected: one `<header>`, one `</header>`, one `header-row`, one `header-text`, one `method-tabs`.

- [ ] **Step 2: Replace the whole header block**

Replace everything from the line `    <header>` through its matching `    </header>` (inclusive) with exactly this. Note: the `.input-bar` markup below is the same node removed in Task 3, reproduced verbatim — its children, IDs, ARIA, and inline styles are unchanged. The `#methodTabs` block keeps every `.method-tab` button verbatim; only the inner `.method-tabs-header` strip is dropped.

```html
    <header>
      <div class="hero">
        <a href="https://4ac.org" class="wordmark" title="4 Awesome Colorizers">
          <span class="wm-4">4</span><span class="wm-ac">ac</span><span class="wm-tld">.org</span>
        </a>
        <div class="eyebrow">4ac = 4 Awesome Colorizers · Four methods · v0.3.7</div>
        <h1>One hex in. A full, accessible <em>palette</em> out.</h1>
        <p class="lede">Primary + 3 accents + status colors + 100–900 ramps. Export-ready. No signup.</p>

        <div class="hero-step" id="heroStep1">Step 1 — your color</div>
        <div class="input-bar">
          <label class="swatch-input" id="swatchInputWrap">
            <input type="color" id="colorInput" value="#5B6CFF">
          </label>
          <div class="hex-field" style="max-width:180px;flex:0 1 auto">
            <label>Input color</label>
            <input type="text" id="hexInput" value="#5B6CFF" maxlength="7" autocomplete="off" spellcheck="false">
          </div>

          <div class="input-divider" aria-hidden="true"><span>OR</span></div>

          <div class="tag-zone" id="tagZone">
            <div class="tag-popover-wrap">
              <button class="tag-trigger" id="tagTriggerBtn" aria-haspopup="true" aria-expanded="false" title="Select Theme to refine the suggested colors">
                <svg class="tag-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"/><line x1="7" y1="7" x2="7.01" y2="7"/></svg>
                <span id="tagTriggerLabel">Select Theme</span>
                <span class="badge" id="tagBadge" style="display:none">0</span>
                <svg class="tag-caret" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><polyline points="6 9 12 15 18 9"/></svg>
              </button>
              <div class="tag-popover" id="tagPopover" role="dialog" aria-label="Pick tags to refine suggestions">
                <div class="tag-popover-header">Pick tags to refine suggestions</div>
                <div class="tag-group">
                  <div class="tag-group-label">Topic — what is this for?</div>
                  <div class="tag-options" id="tagOptionsTopic"></div>
                </div>
                <div class="tag-group">
                  <div class="tag-group-label">Feel — what mood?</div>
                  <div class="tag-options" id="tagOptionsFeel"></div>
                </div>
                <div class="tag-popover-footer">
                  <button class="tag-clear-btn" id="tagClearBtn">Clear all</button>
                  <span class="stats" id="tagStats">No tags</span>
                </div>
              </div>
            </div>
            <div id="tagSelectedChips" style="display:flex;gap:5px;flex-wrap:wrap"></div>
            <div class="tag-conflict-hint" id="tagConflictHint" role="alert"></div>
          </div>

          <div class="presets">
            <button class="preset" style="background:#5B6CFF" data-hex="#5B6CFF" title="Indigo"></button>
            <button class="preset" style="background:#0EA5E9" data-hex="#0EA5E9" title="Sky"></button>
            <button class="preset" style="background:#10B981" data-hex="#10B981" title="Emerald"></button>
            <button class="preset" style="background:#F59E0B" data-hex="#F59E0B" title="Amber"></button>
            <button class="preset" style="background:#EF4444" data-hex="#EF4444" title="Red"></button>
            <button class="preset" style="background:#A855F7" data-hex="#A855F7" title="Purple"></button>
            <button class="preset" style="background:#EC4899" data-hex="#EC4899" title="Pink"></button>
            <button class="preset" style="background:#0F172A" data-hex="#0F172A" title="Slate"></button>
          </div>
        </div>

        <div class="hero-step" id="heroStep2">Step 2 — how the accents are derived</div>
        <div class="method-tabs" id="methodTabs" role="tablist" aria-label="Calculation method">
          <button class="method-tab active" data-method="material" role="tab" aria-selected="true">
            <span class="tab-name">Material</span>
            <span class="tab-desc">Curated premium colors, ranked by fit.</span>
          </button>
          <button class="method-tab" data-method="hsl" role="tab" aria-selected="false">
            <span class="tab-name">HSL Math</span>
            <span class="tab-desc">Classic color theory schemes.</span>
          </button>
          <button class="method-tab" data-method="hybrid" role="tab" aria-selected="false">
            <span class="tab-name">Hybrid</span>
            <span class="tab-desc">HSL math with material safety net.</span>
          </button>
          <button class="method-tab" data-method="oklch" role="tab" aria-selected="false">
            <span class="tab-name">OKLCH-Scored</span>
            <span class="tab-desc">Weighted OKLCH scorer with sliders.</span>
          </button>
        </div>
      </div>
    </header>
```

- [ ] **Step 3: Verify structural invariants (JS contracts intact)**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && \
echo "input-bar: $(grep -c '<div class="input-bar">' index.html) (want 1)" && \
echo "methodTabs: $(grep -c 'id="methodTabs"' index.html) (want 1)" && \
echo "method-tab buttons: $(grep -c 'class="method-tab' index.html) (want 4)" && \
echo "colorInput: $(grep -c 'id="colorInput"' index.html) (want 1)" && \
echo "hexInput: $(grep -c 'id="hexInput"' index.html) (want 1)" && \
echo "presets in input-bar still present: $(grep -c 'class="presets"' index.html) (want 1)" && \
echo "h1 count: $(grep -c '<h1>' index.html) (want 1)" && \
echo "header-row removed: $(grep -c 'class="header-row"' index.html) (want 0)" && \
echo "method-tabs-header removed: $(grep -c 'method-tabs-header' index.html) (want 1 — CSS rule only, removed in Task 5)"
```
Expected: every count matches its "(want N)". `method-tabs-header` still returns 1 here because the dead CSS rule is deleted in Task 5; the markup element is gone.

- [ ] **Step 4: Commit**

```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: single-column layout with Step 1 (input) + Step 2 (methods) inside header"
```

---

### Task 5: CSS — hero column, step labels, method grid, cleanup

Restyle so the moved nodes look right: hero is one centered-width column, step labels match the eyebrow token style, and `.method-tabs` becomes a 4-up responsive grid instead of a 340px right rail.

**Files:**
- Modify: `index.html` CSS — HERO block (~53–64), `.method-tabs`/`.method-tab` (~704–753), responsive (~830–833)

- [ ] **Step 1: Replace the HERO CSS block**

Replace this block (the `/* HERO */` comment through the `.wordmark .wm-tld` rule):

```css
  /* HERO */
  header{padding-bottom:40px;border-bottom:1px solid var(--line);margin-bottom:40px}
  .header-row{display:flex;gap:36px;align-items:flex-start;justify-content:space-between;flex-wrap:wrap}
  .header-text{flex:1;min-width:0;max-width:680px}
  .eyebrow{font-family:var(--mono);font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--muted);margin-bottom:14px}
  h1{font-size:clamp(34px,5vw,56px);line-height:1.05;letter-spacing:-.02em;font-weight:600;margin-bottom:14px}
  h1 em{font-style:normal;background:linear-gradient(105deg,var(--accent-grad-a),var(--accent-grad-b));-webkit-background-clip:text;background-clip:text;color:transparent;transition:background .3s ease}
  .lede{font-size:18px;color:var(--muted);max-width:680px;line-height:1.5}
  .wordmark{display:inline-flex;align-items:baseline;gap:1px;font-size:26px;font-weight:800;letter-spacing:-.04em;line-height:1;margin-bottom:18px;text-decoration:none}
  .wordmark .wm-4{background:linear-gradient(105deg,var(--accent-grad-a,#5B6CFF),var(--accent-grad-b,#A78BFA));-webkit-background-clip:text;background-clip:text;color:transparent}
  .wordmark .wm-ac{color:var(--fg)}
  .wordmark .wm-tld{font-size:13px;font-weight:500;color:var(--muted);margin-left:4px;letter-spacing:0}
```

With:

```css
  /* HERO */
  header{padding-bottom:40px;border-bottom:1px solid var(--line);margin-bottom:40px}
  .hero{display:flex;flex-direction:column;max-width:760px;min-width:0}
  .eyebrow{font-family:var(--mono);font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--muted);margin-bottom:14px}
  h1{font-size:clamp(30px,4.2vw,46px);line-height:1.08;letter-spacing:-.02em;font-weight:600;margin-bottom:10px}
  h1 em{font-style:normal;background:linear-gradient(105deg,var(--accent-grad-a),var(--accent-grad-b));-webkit-background-clip:text;background-clip:text;color:transparent;transition:background .3s ease}
  .lede{font-size:16px;color:var(--muted);max-width:560px;line-height:1.5;margin-bottom:28px}
  .wordmark{display:inline-flex;align-items:baseline;gap:1px;font-size:24px;font-weight:800;letter-spacing:-.04em;line-height:1;margin-bottom:16px;text-decoration:none}
  .wordmark .wm-4{background:linear-gradient(105deg,var(--accent-grad-a,#5B6CFF),var(--accent-grad-b,#A78BFA));-webkit-background-clip:text;background-clip:text;color:transparent}
  .wordmark .wm-ac{color:var(--fg)}
  .wordmark .wm-tld{font-size:13px;font-weight:500;color:var(--muted);margin-left:4px;letter-spacing:0}
  /* Step labels — same token language as .eyebrow */
  .hero-step{font-family:var(--mono);font-size:10px;letter-spacing:.18em;text-transform:uppercase;color:var(--muted);font-weight:600;margin:0 0 10px}
  .hero #heroStep2{margin-top:24px}
  .hero .input-bar{width:100%;margin-bottom:0}
```

- [ ] **Step 2: Replace the `.method-tabs` / `.method-tab` CSS block with a grid**

Replace this block (the `/* METHOD TABS — vertical stack on the right of the header */` comment through `.method-tab:focus-visible{outline:2px solid var(--fg);outline-offset:2px}`):

```css
  /* METHOD TABS — vertical stack on the right of the header */
  .method-tabs{
    display:flex;flex-direction:column;gap:4px;
    background:var(--card);border:1px solid var(--line);border-radius:var(--radius);
    padding:6px;box-shadow:var(--shadow);
    width:340px;flex-shrink:0;
  }
  /* Header label so users see this as a chooser, not a numbered list of steps */
  .method-tabs-header{
    padding:10px 14px 8px;
    display:flex;align-items:center;gap:8px;
    font-family:var(--mono);font-size:10px;letter-spacing:.18em;text-transform:uppercase;
    color:var(--muted);font-weight:600;
    border-bottom:1px dashed var(--line);
    margin-bottom:4px;
  }
  .method-tabs-header svg{width:13px;height:13px;opacity:.85}
  .method-tab{
    width:100%;position:relative;
    padding:11px 14px 11px 40px;border-radius:10px;border:0;background:transparent;cursor:pointer;
    font-family:var(--mono);font-size:11px;letter-spacing:.1em;text-transform:uppercase;font-weight:500;
    color:var(--muted);text-align:left;
    transition:background .18s ease, color .18s ease, transform .12s ease, box-shadow .18s ease;
    display:flex;flex-direction:column;gap:3px;
  }
  /* Radio-style indicator on the left of each tab — clearly says "pick one" */
  .method-tab::before{
    content:'';
    position:absolute;left:14px;top:14px;
    width:14px;height:14px;border-radius:50%;
    border:1.5px solid var(--muted);
    background:transparent;
    transition:border-color .18s, background .18s, box-shadow .18s, transform .18s;
  }
  .method-tab:hover{
    background:var(--code-bg);color:var(--fg);
    transform:translateX(2px);
    box-shadow:inset 0 0 0 1px var(--line);
  }
  .method-tab:hover::before{border-color:var(--fg)}
  .method-tab.active{background:var(--fg);color:var(--bg);transform:translateX(0)}
  .method-tab.active::before{
    border-color:var(--bg);
    background:var(--bg);
    box-shadow:inset 0 0 0 3px var(--fg), 0 0 0 1.5px var(--bg);
  }
  .method-tab .tab-name{font-weight:600;letter-spacing:.06em;font-size:12px}
  .method-tab .tab-desc{font-family:var(--sans);font-size:10.5px;letter-spacing:.01em;text-transform:none;font-weight:400;opacity:.75;line-height:1.35}
  .method-tab.active .tab-desc{opacity:.85}
  .method-tab:focus-visible{outline:2px solid var(--fg);outline-offset:2px}
```

With:

```css
  /* METHOD TABS — 4-up grid under Step 2 in the hero */
  .method-tabs{
    display:grid;grid-template-columns:repeat(4,1fr);gap:8px;
    width:100%;
  }
  .method-tab{
    position:relative;
    padding:13px 14px;border-radius:10px;border:1px solid var(--line);background:var(--card);cursor:pointer;
    color:var(--muted);text-align:left;
    transition:background .18s ease, color .18s ease, border-color .18s ease, box-shadow .18s ease;
    display:flex;flex-direction:column;gap:5px;
  }
  .method-tab:hover{
    background:var(--code-bg);color:var(--fg);border-color:var(--muted);
  }
  .method-tab.active{
    color:var(--fg);background:var(--card);border-color:transparent;
    box-shadow:inset 0 0 0 1.5px var(--accent-grad-b,#A78BFA);
  }
  .method-tab .tab-name{
    font-family:var(--mono);font-size:12px;font-weight:600;letter-spacing:.06em;text-transform:uppercase;
    color:var(--fg);display:flex;align-items:center;gap:8px;
  }
  .method-tab .tab-name::before{
    content:'';width:8px;height:8px;border-radius:50%;flex:none;
    background:linear-gradient(105deg,var(--accent-grad-a,#5B6CFF),var(--accent-grad-b,#A78BFA));
    opacity:.4;transition:opacity .18s ease;
  }
  .method-tab.active .tab-name::before{opacity:1}
  .method-tab .tab-desc{
    font-family:var(--sans);font-size:11.5px;letter-spacing:.01em;font-weight:400;
    opacity:.8;line-height:1.4;color:var(--muted);
  }
  .method-tab.active .tab-desc{opacity:1}
  .method-tab:focus-visible{outline:2px solid var(--fg);outline-offset:2px}
  @media(max-width:640px){
    .method-tabs{grid-template-columns:1fr 1fr}
  }
```

- [ ] **Step 3: Fix the responsive header rules**

Find this block (around line 830):

```css
    .header-row{flex-direction:column;gap:24px}
```
and the nearby:
```css
    .method-tabs{width:100%}
```

`.header-row` no longer exists and `.method-tabs` is already `width:100%`. Replace the line:
```css
    .header-row{flex-direction:column;gap:24px}
```
with:
```css
    .hero{max-width:none}
```
And delete the now-redundant line:
```css
    .method-tabs{width:100%}
```
(If `.method-tabs{width:100%}` is part of a multi-selector rule rather than its own line, leave that rule alone and only change the `.header-row` line.)

- [ ] **Step 4: Verify CSS invariants**

Run:
```bash
cd /c/Users/hicha/.vscode/4ac-org && \
echo "header-row CSS gone: $(grep -c '\.header-row' index.html) (want 0)" && \
echo "header-text CSS gone: $(grep -c '\.header-text' index.html) (want 0)" && \
echo "method-tabs-header gone: $(grep -c 'method-tabs-header' index.html) (want 0)" && \
echo "hero-step styled: $(grep -c '\.hero-step{' index.html) (want 1)" && \
echo "method grid: $(grep -c 'grid-template-columns:repeat(4,1fr)' index.html) (want 1)"
```
Expected: all counts match. (`.top-row-left .input-bar` rule at ~316 is now dead but harmless — leave it; removing it risks nothing and adds churn.)

- [ ] **Step 5: Commit**

```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: CSS for single-column layout, step labels, 4-up method grid"
```

---

### Task 6: Manual browser verification

No automated tests exist; verify the live page behaves.

**Files:** none (verification only)

- [ ] **Step 1: Serve the file**

```bash
cd /c/Users/hicha/.vscode/4ac-org && python3 -m http.server 8080
```
Open `http://localhost:8080` in a browser. (Stop the server with Ctrl-C when done.)

- [ ] **Step 2: Desktop checklist at ~1280×800**

Confirm all true:
- Hero shows: wordmark `4ac.org`, eyebrow reading `4ac = 4 Awesome Colorizers · Four methods · v0.3.7`, headline `One hex in. A full, accessible palette out.` (with `palette` in the accent gradient), the sub line, **Step 1 — your color** above the input bar, **Step 2 — how the accents are derived** above a 4-column method grid.
- Input bar, hex field, color swatch, OR divider, Select-Theme tag popover, and the 8 preset chips all render inside the hero and are above the fold.
- Typing/changing the hex (or clicking a preset, or the color swatch) updates the palette output below.
- The Select-Theme popover opens, tags toggle, "Clear all" works.
- Clicking each of the 4 method tabs switches the active state and changes the palette/panel below; the active tab shows the accent ring.

- [ ] **Step 3: Responsive checklist at ≤640px width**

- Method grid collapses to 2 columns; no horizontal scrollbar; nothing clipped.
- Input bar wraps cleanly (existing mobile rules still apply).
- The existing `.mobile-warning` note still appears as before.

- [ ] **Step 4: Accessibility spot-check**

- Keyboard: Tab reaches the method tabs and they are operable with Enter/Space; focus outline visible.
- Exactly one `<h1>` in the document (`grep -c '<h1>' index.html` → `1`).
- `#methodTabs` still has `role="tablist"` and each tab `role="tab"` with `aria-selected` toggling.

- [ ] **Step 5: Final commit (only if any fix was needed)**

If Steps 2–4 surfaced a defect, fix it, re-verify, and:
```bash
cd /c/Users/hicha/.vscode/4ac-org && git add index.html && git commit -m "Hero: fix <describe defect> found in browser verification"
```
If everything passed with no changes, no commit is needed — the feature is complete.

---

## Self-Review

**Spec coverage:**
- Two labeled steps (Step 1 input / Step 2 methods) → Tasks 4 & 5.
- Outcome headline + sub copy → Task 2.
- `4ac = 4 Awesome Colorizers` visible in one place (eyebrow), replace "Amazing" → Task 1.
- No generate button → confirmed: the reproduced `.input-bar` markup in Task 4 contains none.
- Restructure not duplicate; preserve IDs/JS → Task 3 (detach) + Task 4 (reattach same node verbatim) + Task 4 Step 3 invariant grep; JS selectors `.method-tab` (global) and `.input-bar .presets` (child preserved) verified safe.
- Single column, no `.header-row` split, method grid 4→2up, step-label token style → Task 5.
- ARIA/single-h1 preserved → Task 4 markup + Task 6 Step 4.
- No new deps / no web fonts / no motion added / single file → nothing in any task introduces these.
- Out-of-scope items (toolbar, weights, wheel, README) → untouched by every task.

**Placeholder scan:** No TBD/TODO; every edit shows exact before/after; all grep commands have expected output. None found.

**Type/name consistency:** Class/ID names used consistently across tasks — `.hero`, `.hero-step`, `#heroStep2`, `.input-bar`, `#methodTabs`, `.method-tab`, `--accent-grad-a/b` match between markup (Task 4) and CSS (Task 5). `.method-tabs-header` markup removed in Task 4, its dead CSS removed in Task 5 (Task 4 Step 3 explicitly notes the count stays 1 until Task 5) — consistent, not a contradiction.

No gaps found.
