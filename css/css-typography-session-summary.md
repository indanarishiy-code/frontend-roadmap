# Session Summary: Typography (Section 7)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `font-family` and font stacks

```css
body { font-family: "Inter", "Helvetica Neue", Arial, sans-serif; }
```
A font stack is a fallback chain — browser tries each in order. **The final generic keyword (`sans-serif`, `serif`, `monospace`) is required as a safety net**, not decoration — ensures some reasonable font renders even if every named font fails.

**System font stack pattern (used by GitHub and others):**
```css
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
```
Deliberately uses each OS's native system font instead of a custom webfont. **Why:** zero font download cost (instant render, no flash of invisible/fallback text), UI feels visually native to whatever OS it runs on. A real, deliberate architectural choice to avoid `@font-face` loading complexity entirely — not just "the cheap way out."

## Do frameworks provide font-family fallbacks by default? (verified via search)

**No.** React, Vue, and Angular have zero opinion on typography — a fresh app with no CSS added renders in the browser's default font (typically Times New Roman/system serif), same as raw unstyled HTML.

**What actually provides font stacks in practice:**
- **UI component libraries** (Vuetify, MUI, Ant Design, PrimeVue) — ship their own CSS reset/base styles including font-family defaults. Vuetify specifically applies opinionated base styles via its reset.
- **CSS resets/normalize.css** — a widespread, deliberate manual practice; developers add a reset file explicitly setting `font-family` on `body`.
- **Meta-framework starter templates** (Nuxt, Next.js, Create React App) — some include base CSS with sensible defaults, but this is the template's choice, not the framework's.

**Practical takeaway:** if a past project's typography "just looked fine" without explicitly setting a font stack, it's almost certainly because a UI library or boilerplate already had one set — not because Vue did it invisibly.

## `@font-face` — custom font loading

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;
}
```

**`font-display` values:**
```css
font-display: auto;     /* browser default, varies */
font-display: block;    /* brief invisible period, then swap (FOIT) */
font-display: swap;     /* fallback shown IMMEDIATELY, swaps once loaded (FOUT) */
font-display: fallback; /* very brief invisible period, then permanent fallback if not ready fast enough */
font-display: optional; /* browser decides based on connection speed */
```
**`swap` is the overwhelmingly common practical choice** — text visible immediately using the fallback font beats invisible text while waiting, even with a visible swap later. Same underlying reasoning as FOUC/render-blocking CSS from earlier sessions — visible-but-unstyled beats genuinely invisible.

## Core text properties

```css
font-size: 16px;
font-weight: 400;   /* 100-900, or normal (400)/bold (700) */
line-height: 1.5;    /* unitless — senior-recommended */
letter-spacing: 0.02em;
```

**Unitless `line-height` vs unit-based — important distinction:**
```css
.parent { font-size: 20px; line-height: 1.5; }  /* GOOD: inherited as a RATIO, recalculated per child's own font-size */
.parent { font-size: 20px; line-height: 30px; } /* RISKY: child inherits literal 30px regardless of its own font-size */
```
Unitless is correct because it avoids line-height looking wrong when nested elements have different font sizes than their parent.

## Variable fonts — `font-variation-settings`

```css
@font-face {
  font-family: "Inter Variable";
  src: url("/fonts/inter-variable.woff2") format("woff2-variations");
  font-weight: 100 900; /* a RANGE, not a fixed value */
}
.text { font-variation-settings: "wght" 550; }
```
**Why it matters:** traditional web fonts need a separate file per weight/style (4 weights = 4 files). A single variable font file contains the entire range (weight, sometimes width/slant/optical size), enabling `font-weight: 550` (not even a named weight) with zero additional downloads. Real, current performance technique — fewer network requests, smaller total payload than multiple static files combined, despite one variable file being larger than any single static weight file alone.

## Text properties

```css
text-align: left | right | center | justify | start | end;
text-decoration: underline;
text-transform: uppercase | lowercase | capitalize;
text-overflow: ellipsis;
white-space: nowrap | pre | pre-wrap;
```

**`text-overflow: ellipsis` — common gotcha, needs all three properties together:**
```css
.truncate {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
```
Missing any one and the ellipsis silently fails — a common real debugging pattern worth memorizing as a full set.

**`text-align: start`/`end` vs `left`/`right` — i18n-aware choice:** like `margin-inline-start` from earlier sessions, `start`/`end` automatically flip for RTL languages (Arabic, relevant to UAE target market); `left`/`right` stay physically fixed regardless of direction. Default to `start`/`end` in new code.

## `text-wrap: balance` / `pretty` — recent, worth knowing

```css
h1 { text-wrap: balance; }
p { text-wrap: pretty; }
```
**Problem solved:** headings often wrap with an orphaned single word on the last line or uneven line lengths. `balance` distributes text across lines as evenly as possible — intended for headings/short text blocks. `pretty` is optimized for body paragraph text, specifically avoiding single-word orphan lines.

**Practical constraint:** `balance` has a browser-imposed line-count limit (works well for short headings, not long paragraphs) — computing optimal balance across many lines is computationally expensive. A real, current capability with genuine practical limits.

Visual comparison demonstrated: default (`normal`) wrapping can leave an awkward short/lonely last line; `balance` computes better break points for more even lines; `pretty` avoids orphaned words in flowing paragraph text. Different heuristics for different content types — headings vs body text — which is why they're separate values.

## Where we left off

Section 7 (Typography) covered in full, including a verified (via search) answer on whether frameworks provide font-family defaults (they don't — component libraries/resets do), and a visual demonstration of `text-wrap: balance` vs `pretty` vs default wrapping.

Next up per the CSS reference doc: **Section 8 — Color & Visual Effects** (`background` sub-properties, gradients, `box-shadow`/`text-shadow`/`filter`/`backdrop-filter`, `opacity` vs alpha channel, `border-radius`, `clip-path`, blend modes).
