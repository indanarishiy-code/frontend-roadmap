# Session Summary: CSS Syntax & Units (Section 1, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: from this session onward, explanations are calibrated to senior-frontend depth by default, rather than starting basic.*

## Rule structure

```css
p {
  color: red;
  font-size: 16px;
}
```
- **Selector** (`p`) — what element(s) targeted
- **Declaration block** — everything inside `{ }`
- **Declaration** (`color: red;`) — one property-value pair, ends in semicolon
- **Property** (`color`) / **Value** (`red`)

Multiple selectors (comma-separated) apply the same declarations to each:
```css
h1, h2, h3 { color: navy; margin-bottom: 1rem; }
```

## Comments

```css
/* This is a comment */
```
Only `/* */` syntax — no `//` single-line comments in real CSS (Sass supports `//` but compiles it away before reaching actual CSS).

## Case sensitivity

- Property names/most values: case-insensitive in practice, lowercase is universal convention
- Element selectors: case-insensitive (matches HTML's case-insensitivity), lowercase is convention
- **Class/ID selectors ARE case-sensitive** — `.myClass` ≠ `.myclass`, since they match `class`/`id` attribute values (arbitrary case-sensitive strings)
- `url()` values, custom property names: case-sensitive (often reference actual file paths/identifiers)

## Units — absolute

**`px`** — reference pixel, not literal device pixel; defined relative to "reasonable viewing distance" so it looks visually similar across devices with different physical pixel densities. This abstraction is what makes responsive design possible at all.

**When to use `px`:** borders (`border: 1px solid`, avoids inconsistent thickness at different zoom/font settings), box-shadow offsets/blur (shadow physics don't need to scale with text), some fixed icon sizing (increasingly debated — many teams now prefer `rem` even here).
**When NOT to use:** font-size/spacing that should respect accessibility preferences — a common senior-review flag.

**`pt`/`cm`/`in`/`mm`** — genuinely only relevant for `@media print` (page margins, print layout dimensions where physical measurement matters). Rare/questionable outside print context.

## Units — relative

**`%`** — relative to parent's corresponding property. Gotcha: `padding`/`margin` percentages (even `padding-top`/`padding-bottom`) are always calculated relative to parent's **width** — this was deliberately exploited for the aspect-ratio-box hack before native `aspect-ratio` existed.
- **Use for:** fluid width layouts, `max-width: 100%` on images (most common real use, prevents overflow)
- **Avoid for:** `height: %` unless parent has explicit height — silently does nothing if parent height is `auto` (common beginner bug)

**`em`** — relative to the **current element's own font-size** (not parent — common misconception). Compounds when nested elements each use `em` for both font-size and padding.
```css
.parent { font-size: 20px; }
.child { font-size: 1.5em; padding: 1em; } /* font-size: 30px, padding: 30px based on child's OWN new font-size */
```
- **Use for:** component-internal scaling (padding scales with that component's own font-size intentionally), icon sizing relative to adjacent text (`width: 1.2em`)
- **Avoid for:** page-wide spacing systems — compounding makes it unpredictable at scale

**`rem`** — relative to root (`<html>`) font-size, always, regardless of nesting. Avoids em's compounding problem while still respecting user font-size preferences (accessibility: browser font-size bump scales rem-based layouts, not hardcoded px).
- **Use for:** the default choice for almost everything in production — font-size, margin, padding, gap, border-radius, spacing-scale design tokens (`--space-sm: 0.5rem`)

**Viewport units — `vw`/`vh`/`vmin`/`vmax`** — percentages of viewport dimensions. `vmin`/`vmax` resolve to whichever of `vw`/`vh` is smaller/larger.
- **`vw`/`vh` use:** full-viewport hero sections (`height: 100vh`), fluid typography (`font-size: 4vw`, usually combined with `clamp()` to avoid extremes)
- **`vmin` use:** square/constrained elements that must fit the smaller viewport dimension regardless of orientation (`width: 80vmin` modal/avatar)
- **`vmax` use:** rare — scaling with whichever dimension is larger (e.g. decorative background element)

**Mobile viewport gotcha (senior-level, current):** `100vh` on mobile doesn't account for the browser's dynamically appearing/disappearing address bar, causing cut-off content or unexpected scrollbars. Solved by **dynamic viewport units:**
- **`dvh`/`dvw`** (dynamic — adjusts live as browser chrome shows/hides) — use for full-screen mobile UI (nav drawers, full-screen modals) needing to reliably match visible area
- **`svh`/`svw`** (small viewport — assumes chrome always visible) — "worst case" design
- **`lvh`/`lvw`** (large viewport — assumes chrome always hidden) — "best case" design

**`ch`** — relative to the width of the "0" character in current font.
- **Use for:** constraining text-block width for readability (`max-width: 65ch` — well-documented UX best practice for optimal line length), fixed-width monospace/character-count-based sizing (`width: 20ch` for a code input)

**`ex`** — relative to font's x-height. Practically never used in modern frontend work — legacy/print-typography recognition only.

## The senior-level takeaway

Unit choice is itself a design decision communicating intent:
- `rem` — "respect user preferences, stay predictable regardless of nesting"
- `em` — "scale with local component context"
- `%`/viewport units — "respond to container/viewport"
- `ch` — "about readable text measure"

This is the same category of decision as choosing `<section>` vs `<div>` in HTML — not arbitrary, purpose-driven.

## Where we left off

Section 1 (Syntax & Core Concepts) in progress: rule structure, comments, case sensitivity, and units (absolute + relative, including current dynamic viewport unit gotchas) covered at senior-frontend depth. Remaining in Section 1: color formats (hex, `rgb()`/`rgba()`, `hsl()`/`hsla()`, named colors, `oklch()`, `color-mix()`) and the `!important` keyword.
