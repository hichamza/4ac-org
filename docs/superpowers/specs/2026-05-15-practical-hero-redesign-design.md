# Practical Hero Redesign — Design

**Date:** 2026-05-15
**Scope:** Hero + first impression only (`index.html`). Everything below the input/method
region (weights, panels, color wheel, palette output, toolbar behavior) is out of scope.

## Goal

Make the first impression **practical and obvious**, not decorative. A first-time visitor
should, without scrolling or interpreting marketing copy, immediately understand:

1. What the tool produces (one hex → full accessible palette).
2. What to do (enter a color, pick a method).

Explicitly **not** a goal: ambient motion, a "living" ribbon, or a premium/editorial mood.
Those directions were explored and rejected by the user in favor of clarity.

## Decisions (locked)

- **Layout:** Two visible, labeled steps inside the hero:
  - **Step 1 — your color:** the existing input bar.
  - **Step 2 — how the accents are derived:** the four methods as a labeled grid.
- **Headline:** outcome-stated, plain language. Working copy:
  - H1: `One hex in. A full, accessible palette out.`
  - Sub: `Primary + 3 accents + status colors + 100–900 ramps. Export-ready. No signup.`
- **Acronym requirement:** `4 Awesome Colorizers` must be visibly present in the
  hero eyebrow. Replaces the current "4 Amazing Colorizers" wording. (Revised
  2026-05-15: the earlier `4ac = 4 Awesome Colorizers` form was dropped — the
  repeated "4" and the `=` read awkwardly; just "4 Awesome Colorizers".)
- **No generate button.** The tool is live/instant; the palette below updates as input
  changes. The mockup's "Generate →" button was illustrative only and must NOT be added.
- **No new dependencies, no web fonts, single file.** Consistent with the project's
  zero-dependency constraint (`README.md`).

## Approach: restructure, don't duplicate

The hero is rebuilt by **re-parenting and restyling existing elements**, not by adding new
input/method markup. This preserves every existing ID and all JS wiring.

Current DOM (relevant slice of `index.html`):

```
header
  .header-row
    .header-text   → .wordmark, .eyebrow, h1, .lede
    .method-tabs#methodTabs   → 4 × .method-tab[data-method] (+ header strip)
.toolbar           → Share / Export / Reset
.top-row > .top-row-left
  .input-bar       → #colorInput, #hexInput, OR divider, .tag-zone, .presets
  .weights / .method-panel[...]   (OUT OF SCOPE — leave in place)
```

Target DOM:

```
header
  .wordmark + acronym tagline ("4 Awesome Colorizers")
  h1 (new outcome copy)  +  .lede/sub
  .hero-step "Step 1 — your color"
  .input-bar              ← MOVED here, contents unchanged
  .hero-step "Step 2 — how the accents are derived"
  .method-tabs#methodTabs ← MOVED here, restyled into a 4-up grid
.toolbar  (unchanged, stays after header)
.top-row > .top-row-left
  .weights / .method-panel[...]   (unchanged)
```

### Units of work

1. **Wordmark + acronym** — keep `.wordmark` markup unchanged. The acronym expansion
   "4 Awesome Colorizers" appears in exactly one place: the existing `.eyebrow` line,
   which currently reads "4 Amazing Colorizers · Four methods · v0.3.5" → becomes
   "4 Awesome Colorizers · Four methods · v0.3.7". Do not add a second tagline element.
   `<title>`/meta currently use "Color pairing, your method of choice." (no "Amazing"
   string) so no meta change is required; if grep finds "Amazing" anywhere in `index.html`
   it is updated to "Awesome" (copy only). Interface: visual only.
2. **Headline copy** — replace `<h1>` text and `.lede` text with the locked copy. Keep the
   `<em>`-gradient treatment available but optional; plain is acceptable.
3. **Step labels** — new `.hero-step` element (mono, uppercase, muted — reuse `.eyebrow`
   token values). Two instances. Purely presentational.
4. **Move `.input-bar` into `header`** — relocate the existing `.input-bar` node from
   `.top-row-left` into the hero, directly under Step 1. No change to its children
   (`#colorInput`, `#hexInput`, OR divider, `.tag-zone`, `.presets`) so all listeners keep
   working. `.top-row-left` then contains only the weights/method panels.
5. **Move + restyle `.method-tabs`** — relocate `#methodTabs` from `.header-row` into the
   hero under Step 2. Restyle from the current right-rail card into a responsive grid:
   4 columns desktop → 2 columns ≤640px. Keep each `.method-tab[data-method]`, its
   `.tab-name` and `.tab-desc`, `role="tab"`/`aria-selected`, and the active-state class
   so the existing method-switch JS is untouched. The "Pick a method · click to switch"
   header strip is replaced by the Step 2 label.
6. **Header layout CSS** — `.header-row` flex split is removed; hero becomes a single
   vertical column constrained to the container width. Remove now-unused right-rail rules
   for `.method-tabs` placement; add grid rules.

### Constraints / invariants

- No JS logic changes. Only DOM position and CSS. If any selector in JS depends on
  `.method-tabs` being a child of `.header-row` or `.input-bar` being in `.top-row-left`,
  the move must preserve the element identity (same IDs/classes) so query selectors by
  ID/class still resolve. Verify by grep before moving.
- `prefers-reduced-motion`: no new motion is introduced, so nothing to add.
- Mobile: the existing `.mobile-warning` note and responsive behavior below the hero are
  unchanged. The new method grid must collapse to 2-up at ≤640px.
- Accessibility: preserve `role="tablist"`/`role="tab"`/`aria-selected` on the method
  tabs; keep a single `<h1>`; step labels are decorative text, not headings.

## Testing / verification

Manual, in-browser (no test framework in repo):

1. Page loads; hero shows wordmark + "4 Awesome Colorizers", outcome headline, Step 1
   input bar, Step 2 method grid — all above the fold at 1280×800.
2. Hex input + color swatch + presets + tag selector still function (palette below
   updates).
3. Clicking each method tab still switches the algorithm and updates active state.
4. ≤640px: method grid is 2-up; no horizontal overflow; nothing clipped.
5. Keyboard: tabs reachable and operable; focus visible; one `<h1>` in the document.
6. `git grep` confirms no JS selector relied on the old parent containers.

## Out of scope

Toolbar redesign, weights/panel styling, palette card styling, color wheel, export menu,
copy below the hero, analytics, the `palette-algorithm.html` page.
