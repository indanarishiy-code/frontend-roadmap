# Session Summary: Responsive Design (Section 6)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Media queries — beyond just width

```css
@media (min-width: 768px) { .container { flex-direction: row; } }
@media (orientation: landscape) { }
@media (hover: hover) and (pointer: fine) { }
```
**`(hover: hover) and (pointer: fine)`** — detects an actual mouse/trackpad (precise pointer, real hover capability) vs touch devices where `:hover` gets "stuck" after a tap (no real hover concept). Practical use: disabling hover-triggered dropdowns specifically on touch devices, where tap-to-open makes more sense.

## `prefers-color-scheme`

```css
@media (prefers-color-scheme: dark) { body { background: #1a1a1a; color: #eee; } }
```
Detects the user's **OS-level** dark/light mode preference (system settings, not per-site config) — lets a site respect it automatically, often alongside a manual in-app override.

**Relevance for dashboards specifically:** genuinely high — dashboards are long-session-duration tools (developers/analysts staring at them for hours) where dark mode is commonly *expected*, not just nice-to-have. Real products (Grafana, Vercel, Linear) ship it as first-class for this reason.

## `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```
Detects an **OS-level accessibility setting** some users enable because animation/motion can cause real physical symptoms (dizziness, nausea, migraines) for people with vestibular disorders — a documented accessibility need, not a stylistic preference.

**Relevance for dashboards:** if the dashboard has animated transitions (chart animations, sliding panels, elaborate loading spinners), respecting this is a genuine accessibility requirement — ties directly to the EAA compliance context covered earlier, since motion-related failures are a real documented complaint category.

## Mobile-first vs desktop-first — and whether it applies to dashboard/internal tool work

```css
/* Mobile-first (min-width — modern standard) */
.card { padding: 8px; }
@media (min-width: 768px) { .card { padding: 16px; } }
```
**Why mobile-first won generally:** base styles target the simplest/most constrained case, complexity is added progressively — usually produces simpler, more maintainable CSS, and aligns with mobile traffic dominating most public products.

**Honest answer for dashboard/CMS/admin work specifically: mobile-first is NOT required in the same way.** Reasons:
- Users are almost always on laptop/desktop with a mouse
- Complex data tables and dense multi-panel layouts are often genuinely difficult or impossible to meaningfully compress to mobile width without losing core functionality
- Mobile access, if it happens, is usually a secondary "quick glance" use case, not primary interaction

**Practical takeaway:** building desktop-first for dashboard-style tools is completely reasonable, adding breakpoints only where layouts would genuinely break (e.g. tablet). Mobile-first is a best practice for a specific problem (public content sites with heavy mobile traffic), not a universal law. **Interview framing:** the strong answer isn't "I always do mobile-first" — it's "I choose the approach based on the product's actual usage pattern."

## Container queries — genuine paradigm shift, flagged recent gap

```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}
@container card (min-width: 400px) {
  .card { flex-direction: row; }
}
```

**Why this matters architecturally:** media queries respond to **viewport** size — blind to how much space a component actually has in its specific placement context. A card component might need different styling in a wide main area vs a narrow sidebar; media queries can't distinguish this (only know browser window size).

**Container queries let a component respond to its own container's size**, regardless of viewport — same component genuinely responsive wherever placed (sidebar, wide grid cell, narrow modal), no separate variant classes or hand-rolled JS `ResizeObserver` logic needed.

**`container-type` values:**
```css
container-type: inline-size;  /* query inline (horizontal) dimension only — used the vast majority of the time */
container-type: size;         /* query both dimensions — more expensive, real layout constraints */
container-type: normal;       /* default — not a query container */
```
`inline-size` is preferred because height-based container queries have real technical limitations (container height often depends on content — circular dependency risk).

**Container query units:**
```css
.card-title { font-size: 5cqi; } /* 5% of container's inline size */
```
`cqw`/`cqh`/`cqi`/`cqb` — container-relative equivalents of `vw`/`vh`, for truly component-scoped responsive typography.

**Architectural significance:** enables genuinely reusable, drop-anywhere components (a card that looks right in a 3-column grid or a full-width hero) without the component needing to know where it's used — a real, current best-practice shift in component library design, directly relevant to Vue component work.

## `clamp()`, `min()`, `max()` — fluid sizing without breakpoints

```css
font-size: clamp(MIN, PREFERRED, MAX);
```

**Worked example:**
```css
h1 { font-size: clamp(1.5rem, 4vw, 3rem); }
```
- Narrow screen (375px): `4vw` ≈ 0.94rem — below min → clamped UP to `1.5rem`
- Mid screen (1000px): `4vw` = 2.5rem — within bounds → uses preferred value directly, `2.5rem`
- Huge screen (2000px): `4vw` = 5rem — exceeds max → clamped DOWN to `3rem`

Scales smoothly between bounds, never exceeding either end regardless of screen size extremity.

**Spacing example:**
```css
.card { padding: clamp(1rem, 2vw, 2.5rem); }
```
Same logic — grows with viewport but stays within a sane range.

**Standard real pattern — mixing `rem` and `vw` in the preferred value:**
```css
font-size: clamp(1rem, 0.5rem + 2vw, 2rem);
```
The `rem` base ensures a sane floor even at `0vw`; the `vw` component adds actual scaling. More robust than pure `vw` alone, which can produce awkwardly small values requiring the min-clamp to work harder to correct.

**Why this beats media queries for typography specifically:** without `clamp()`, smooth typography scaling requires either accepting abrupt fixed-size jumps at breakpoints, or many closely-spaced media queries to fake smoothness (verbose, hard to maintain). `clamp()` gives smooth scaling in one line — the standard modern approach for fluid type scales.

```css
width: min(90%, 600px);   /* smaller of the two — caps max size on huge screens */
width: max(200px, 20%);   /* larger of the two — ensures a minimum regardless of container shrinkage */
```

## `@supports` — progressive enhancement

```css
@supports (container-type: inline-size) {
  .card { container-type: inline-size; }
}
@supports not (container-type: inline-size) {
  .card { /* fallback for older browsers */ }
}
```
Conditionally applies CSS based on actual browser feature support rather than browser-sniffing/version checks — the correct native mechanism for providing fallbacks for genuinely new features (container queries, `:has()`, `subgrid`) when supporting older browsers.

## Where we left off

Section 6 (Responsive Design) covered in full, including a genuinely practical discussion tailored to dashboard/internal-tool work specifically (mobile-first isn't a universal requirement, `prefers-color-scheme`/`prefers-reduced-motion` relevance for long-session dashboard use) and detailed `clamp()` worked examples. Container queries flagged as the standout, most architecturally significant piece — directly maps to the roadmap's suggested exercise (rebuilding a component with container queries instead of media queries), not yet attempted hands-on.

Next up per the CSS reference doc: **Section 7 — Typography** (`font-family`/font stacks, `@font-face` and `font-display` strategies, `font-size`/`font-weight`/`line-height`/`letter-spacing`, variable fonts, text properties, `text-wrap: balance`/`pretty`).
