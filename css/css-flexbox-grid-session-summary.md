# Session Summary: Flexbox & Grid (Section 5, Part 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Flexbox — container properties

```css
.container {
  display: flex;
  flex-direction: row;         /* row (default) | row-reverse | column | column-reverse */
  flex-wrap: nowrap;           /* nowrap (default) | wrap | wrap-reverse */
  justify-content: flex-start;  /* main-axis alignment */
  align-items: stretch;         /* cross-axis alignment */
  align-content: normal;        /* cross-axis alignment for WRAPPED lines specifically */
  gap: 16px;
}
```

## Core mental model: main axis vs cross axis

`flex-direction: row` → main axis horizontal, cross axis vertical. `flex-direction: column` → main axis vertical, cross axis horizontal.

**`justify-content` always operates on the main axis. `align-items`/`align-content` always operate on the cross axis.** This is why `justify-content: center` centers horizontally in `row` but vertically in `column` — the property's meaning never changes, the axis it acts on rotates with direction. Resolves most "why isn't my flexbox centering working" confusion.

## `justify-content` — the confusing trio

- `space-between` — no space at start/end, equal space only between items
- `space-around` — equal space around each item, but edges get half the space of between-gaps (each item has space both sides; edges effectively single-space, middles double)
- `space-evenly` — truly equal space everywhere including edges — usually what people actually want when reaching for `space-around`

## `align-items: stretch` (default) — worth calling out

Items stretch to fill the cross axis by default unless they have explicit size or `align-self` override — this is why flex items often match their tallest sibling's height with zero extra CSS, and what makes "equal height columns" trivial in Flexbox (a real hack pre-Flexbox).

## Flex item properties

```css
.item {
  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto;
  flex: 0 1 auto;   /* shorthand: grow shrink basis */
  align-self: auto;  /* override align-items for THIS item */
  order: 0;           /* visual reordering, does NOT change DOM order */
}
```

## `flex: 1` vs `flex: auto` — genuinely important, often-missed distinction

`flex: 1` = `flex: 1 1 0%` — basis is **0%**, meaning content size is ignored as a starting point entirely; space is distributed purely by grow ratio from zero. Multiple `flex: 1` items become genuinely equal width regardless of content length.

`flex: auto` = `flex: 1 1 auto` — content size DOES matter as a starting point; items with more content get more space first, then remaining space distributes by grow ratio.

**Common bug:** "why aren't my `flex: 1` items equal width" almost always traces back to accidentally using `flex: auto` or a non-zero basis somewhere.

## `order` — accessibility caveat

Changes **visual** order only — does NOT change DOM order, tab order, or screen-reader announcement order. Real documented accessibility problem: sighted keyboard users tab in an order mismatched with what they see; screen readers announce original DOM order regardless of visual rearrangement.

**Senior rule:** fine for minor cosmetic reordering; never use it to significantly diverge visual order from reading order — if that's needed, the actual DOM/component structure should be reordered instead.

## Common Flexbox patterns

```css
/* Perfect centering, both axes */
.container { display: flex; justify-content: center; align-items: center; }

/* Equal-height columns (leverages align-items: stretch default) */
.row { display: flex; }

/* Space-between navbar */
.navbar { display: flex; justify-content: space-between; align-items: center; }

/* Sticky footer pattern */
.page { display: flex; flex-direction: column; min-height: 100vh; }
.main-content { flex: 1; } /* pushes footer down regardless of content height */
```

---

## Grid — core mental model shift from Flexbox

Flexbox is **one-dimensional** (single axis, even with wrapping). Grid is **two-dimensional** — rows and columns defined simultaneously, items placed precisely into cells/spans in both dimensions. Not "Grid is better" — genuinely different layout shapes.

## Basic grid setup

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 100px;
  grid-template-rows: 80px auto 60px;
  gap: 16px;
}
```
Children flow into cells in DOM order unless explicitly placed.

## `fr` unit

```css
grid-template-columns: 1fr 2fr 1fr;
```
Represents a fraction of **remaining available space** after fixed-size tracks are subtracted — fundamentally different from `%`, since `fr` automatically accounts for fixed-width siblings. Middle column here gets exactly 2x each outer column.

## `repeat()` and `minmax()` — responsive grid workhorses

```css
grid-template-columns: repeat(3, 1fr);
grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
```
`minmax(200px, 1fr)` — track is at least 200px, grows to fill available space up to 1fr's share.

**`repeat(auto-fill, minmax(200px, 1fr))`** — the single most useful responsive grid pattern in modern CSS: creates as many 200px+ columns as fit, automatically, zero JS/media queries.

**`auto-fill` vs `auto-fit` — the subtle distinction:** `auto-fill` keeps empty tracks (leaves gaps if too few items to fill a row); `auto-fit` collapses empty tracks to zero, letting existing items stretch to fill leftover space. 3 items in a 5-item-capable row: `auto-fit` stretches the 3 to fill; `auto-fill` leaves 2 invisible empty columns.

## `grid-template-areas` — declarative, senior-friendly

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
}
.sidebar { grid-area: sidebar; }
.header  { grid-area: header; }
```
CSS literally reads as an ASCII-art diagram of the layout — genuine maintainability win over mentally reconstructing layout from line numbers.

## `grid-column`/`grid-row` — line-based placement

```css
.item {
  grid-column: 1 / 3;   /* spans line 1 to line 3 — 2 columns wide */
  grid-row: 2 / span 2; /* starts at line 2, spans 2 rows */
}
```
Grid lines numbered from 1, counting track *boundaries* not tracks/cells themselves — trips people up initially. `span` is more intuitive than exact end-line numbers when you just care about track count.

## Implicit vs explicit grid

**Explicit** — tracks defined via `grid-template-columns`/`rows`. **Implicit** — tracks auto-created when content overflows the explicit definition:
```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 100px; /* sizes implicitly-created rows */
}
```
10 items in a 3-column explicit grid → extra rows auto-created (implicit grid); `grid-auto-rows`/`grid-auto-columns` size these since they weren't explicitly defined.

**`grid-auto-flow`:**
```css
grid-auto-flow: row;    /* default — fill rows first */
grid-auto-flow: column; /* fill columns first */
grid-auto-flow: dense;  /* backfill gaps, may change visual order — same accessibility caveat as flex `order` */
```

## `subgrid` — flagged recent gap area

```css
.parent { display: grid; grid-template-columns: repeat(4, 1fr); }
.child {
  display: grid;
  grid-column: 1 / 4;
  grid-template-columns: subgrid; /* inherits PARENT's column tracks, not its own new ones */
}
```
**Real problem solved:** before `subgrid`, a nested grid inside a grid item created its own independent tracks — nested items could never align to the outer grid's lines. Mattered for card layouts where, e.g., every card's "title"/"price" should align vertically across cards despite each card being internally its own grid. `subgrid` lets a nested grid genuinely participate in and align with parent tracks — a real structural capability that didn't exist in CSS until relatively recently.

## Multi-column layout — niche

```css
.article {
  columns: 3;
  column-gap: 32px;
  column-rule: 1px solid #ccc;
}
```
Newspaper-style flowing text columns — content auto-flows column to column, unlike item-based Grid/Flexbox placement. Niche in modern app UI (long-form text/magazine-style content specifically) — not replaced by Grid/Flexbox since automatic text-reflow is a fundamentally different problem than item layout.

## Where we left off

Second slice of Section 5 covered: full Flexbox (container/item properties, `flex: 1` vs `flex: auto`, `order`'s accessibility caveat, common patterns) and full Grid (`fr`, `repeat()`/`minmax()`, `auto-fill` vs `auto-fit`, `grid-template-areas`, line-based placement, implicit vs explicit grid, `subgrid`, multi-column layout). `subgrid` and `repeat(auto-fill/auto-fit, minmax())` flagged as the two most currently load-bearing concepts. No exercise built yet — the roadmap's suggested exercise (rebuilding a component with Grid/`:has()` instead of JS-driven logic) remains a good candidate for later.

This closes out **Section 5: Layout Systems** entirely across two sessions (normal flow/display/positioning/stacking-contexts, then Flexbox/Grid/multi-column).

Next up per the CSS reference doc: **Section 6 — Responsive Design** (media queries, container queries — a flagged recent-gap topic — mobile-first vs desktop-first, `clamp()`/`min()`/`max()`, responsive typography, `@supports`).
