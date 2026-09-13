# Session Summary: Performance-Relevant CSS (Section 16)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: critical rendering path and critical CSS extraction/inlining were already covered in depth in the HTML Performance session — not repeated here.*

## Selector performance — mostly a non-issue today, but worth understanding why

```css
div.container ul li a.link { }  /* historically claimed "slow" */
.link { }                        /* modern reality: negligible difference */
```
**Current answer:** modern browser engines are extremely optimized — selector performance differences that mattered in the 2010s are largely irrelevant today for typical page sizes. A common outdated interview talking point worth correcting rather than repeating.

**Worth understanding regardless:** browsers match selectors **right-to-left**, not left-to-right as most people intuitively assume. For `div.container ul li a.link`, the browser first finds all `a.link` elements, then checks each one's ancestors against the rest of the selector — not starting from `div.container` and working down. This is why extremely generic rightmost selectors (`*`, or an overly broad tag as the final component) can still have measurable cost in truly enormous DOM trees, even though "descendant selectors are slow" is largely outdated for normal-sized pages.

## `contain` — genuinely current, real performance tool

```css
.card {
  contain: layout;   /* internal layout doesn't affect anything outside it */
  contain: paint;     /* nothing inside paints outside its bounds */
  contain: size;       /* size doesn't depend on children */
  contain: content;    /* shorthand: layout + paint */
  contain: strict;      /* shorthand: layout + paint + size */
}
```

**What it does:** normally, when content changes anywhere in the DOM, the browser may need to recalculate layout/paint for a potentially large portion of the page, since CSS's cascading, context-dependent nature means changes *can* ripple outward. `contain` is an explicit promise to the browser: "changes inside this element will never affect anything outside its boundary" — lets the browser skip recalculating everything outside the contained element, genuinely reducing the scope of expensive layout/paint work.

**Concrete practical scenario — directly relevant to dashboard work:** a dashboard with many independent card widgets, each containing frequently-updating dynamic data (live metrics, charts). Without `contain`, every update inside one card could theoretically require checking whether it affects layout elsewhere on the page. With `contain: layout` on each card, the browser skips that check — a real, measurable performance win for exactly this kind of dashboard UI.

## `content-visibility` — more dramatic, newer capability

```css
.offscreen-section {
  content-visibility: auto;
}
```

**What it does:** tells the browser to **skip rendering (layout, paint, and most accessibility tree work) for content that's currently off-screen**, as if it doesn't exist yet — until it's about to scroll into view, when the browser renders it just in time.

**Key distinction from `loading="lazy"`:** `loading="lazy"` (on images) defers *loading* the resource. `content-visibility` defers the actual *rendering* work for arbitrary DOM content, regardless of whether resources are already loaded — a fundamentally different, broader mechanism.

**Why this is a big deal for long pages/lists:** a page with hundreds of list items, cards, or sections can see dramatically faster initial render times, since the browser skips layout/paint work for content not yet visible. Real, measurable technique for long article pages, large data tables, or infinite-scroll-style dashboards — achieves some of virtual scrolling's (a JS technique) performance benefit with a single CSS property and zero JS.

**Critical companion property — `contain-intrinsic-size`:**
```css
.offscreen-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* estimated size while not yet rendered */
}
```
**The real gotcha:** since `content-visibility: auto` skips rendering unrendered content, the browser doesn't know how tall that content actually is. Without `contain-intrinsic-size` providing a size estimate, unrendered sections collapse to zero height, causing the scrollbar to jump around erratically as content renders in during scroll. Providing an estimated size (even rough) keeps scrollbar behavior stable and prevents this jarring layout shift.

## Where we left off

Focused summary on `contain` and `content-visibility` — the two genuinely new, currently-relevant performance tools in Section 16 (selector performance and critical rendering path/critical CSS were covered as recap/correction of outdated assumptions, with critical rendering path itself already fully covered in the earlier HTML Performance session). `content-visibility` flagged as a particularly significant capability given its direct applicability to long-list/dashboard UI work.

Next up per the CSS reference doc: **Section 17 — Tooling & Preprocessing Context** (Sass/SCSS and what native CSS has replaced vs. what Sass still offers, PostCSS/autoprefixing, Lightning CSS, Vanilla Extract, vendor prefixes).
