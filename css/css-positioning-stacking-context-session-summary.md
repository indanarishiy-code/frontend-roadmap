# Session Summary: Normal Flow, Display, Positioning & Stacking Contexts (Section 5, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. This is the first of multiple planned sessions on Section 5 (Layout Systems), given its size.*

## Normal flow

Default layout behavior before positioning/flex/grid — elements stack according to `display` type and source order, top to bottom.

## Block vs inline vs inline-block

**Block** (`<div>`, `<p>`, `<section>`): full available width by default, always starts a new line, respects `width`/`height`/vertical margin-padding fully.

**Inline** (`<span>`, `<a>`, `<strong>`): only as wide as content, doesn't start a new line, **`width`/`height` ignored entirely**, vertical margin/padding render visually but don't push surrounding layout — a common confusion point (`padding-top` on a `<span>` adds visual space but doesn't move the line above it).

**Inline-block** — hybrid: flows inline (no forced line break) but respects `width`/`height`/full margin-padding like block. Was the primary pre-Flexbox technique for horizontal layouts. **Known quirk:** unwanted whitespace gaps between inline-block elements caused by literal whitespace/line breaks in HTML source (inline-block treats that whitespace as real content) — required hacky fixes (removing HTML whitespace, `font-size: 0` tricks) before Flexbox existed.

## `display` — key values

```css
display: block | inline | inline-block | flex | inline-flex | grid | inline-grid | none | contents;
```

**`display: none`** — removes element entirely from layout AND accessibility tree, as if it doesn't exist. Different from `visibility: hidden` (still takes up space, invisible) and `opacity: 0` (still takes up space, still interactive/focusable unless separately disabled). None of these three is correct for "screen-reader-only but visually hidden" text — that needs a specific clipping technique (Section 14).

**`display: contents`** — element disappears from the layout box tree entirely, but its children render as if direct children of the element's parent. The element still exists in DOM/accessibility tree, just generates no box of its own.
```html
<div class="wrapper" style="display: contents;">
  <span>A</span><span>B</span>
</div>
```
`.wrapper` contributes zero layout box — A and B behave as direct siblings of whatever contains `.wrapper`. **Use case:** a wrapping element needed for JS/semantic/structure reasons that's breaking Grid/Flexbox by inserting an unwanted extra layout level.

**Known gotcha:** `display: contents` has documented accessibility bugs in some browser/screen-reader combinations — historically could lose accessible name/role info for children, not just the element's own box. Improved but worth testing directly if accessibility-sensitive.

## `display: contents` vs Vue `<template>` (multi-root) vs React `<Fragment>` — genuinely different mechanisms

| | Fragment / Vue multi-root | `display: contents` |
|---|---|---|
| Does the wrapper exist in the DOM at all? | **No** — never created | **Yes** — real DOM node |
| Can JS query/reference it? | No — nothing to query | Yes — `querySelector` still works |
| Can it have event listeners, `ref`, ARIA attributes? | No | Yes — real element, just no layout box |
| Layer it operates at | JavaScript/component compilation | CSS rendering |

**Fragment/multi-root** solves a **JS-framework problem**: React/older Vue required a single root element from a component; Fragment/`<template>` multi-root lets a component render multiple top-level elements with **zero wrapper element created at all** — not even one that gets hidden.

**`display: contents`** solves a **CSS-only problem**: the wrapper genuinely exists in the DOM (for a real JS reason — a `ref`, event listener, conditional class toggle) but shouldn't participate in the parent's layout (Grid/Flexbox).

**Practical rule:** if the wrapper exists purely to satisfy "component must have one root" syntax with zero other use — use Fragment/multi-root (solves it at the source). If there's a genuine JS reason to need a real DOM node (ref, listener) but it's breaking a Grid/Flex layout — `display: contents` is the right tool.

## Positioning — the five `position` values

```css
position: static | relative | absolute | fixed | sticky;
```

**`relative`** — offsets visually from where it would normally sit, but **still occupies its original space** in layout (a visible "hole" remains, other elements don't reflow). **Most common legitimate use:** `position: relative` with NO offset at all, purely to establish a positioning context for an absolutely-positioned child:
```css
.card { position: relative; }
.card .badge { position: absolute; top: 0; right: 0; } /* positions relative to .card */
```

**`absolute`** — positions relative to the nearest ancestor with any non-`static` position; falls back to the whole initial containing block (~viewport) if no such ancestor exists. **Common bug:** forgetting to set `position: relative` on the intended parent, causing the absolute element to jump relative to the whole page instead. Removed entirely from normal flow — takes up no space, other elements act as if it doesn't exist.

**`fixed`** — relative to the viewport, ignores scroll. **Senior gotcha:** breaks if any ancestor has `transform`, `filter`, `perspective`, or `will-change` set — these create a new containing block, and `fixed` positions relative to *that* ancestor instead of the viewport. Common confusing bug: a `fixed` element suddenly scrolls with the page after an unrelated ancestor gets a `transform` for an animation.

**`sticky`** — hybrid: `relative` until a scroll threshold (`top: 0`), then behaves like `fixed`, but only within its containing parent's bounds — scrolls away once the parent leaves view (unlike true `fixed`). **Common gotcha:** silently doesn't work if any ancestor has `overflow: hidden`/`scroll`/`auto` — breaks the scroll-tracking calculation sticky needs.

## `z-index` and stacking contexts — the genuinely deep part

Higher `z-index` stacks on top — simple in isolation. Real complexity: **a stacking context is a self-contained "layer group"** — once established, `z-index` values inside are only compared against each other, never directly against values outside it.

**Elements that create a new stacking context:** any positioned element with `z-index` other than `auto`, `opacity < 1`, `transform`, `filter`, `will-change`, `isolation: isolate`, flex/grid items with a `z-index`.

**Classic confusing bug:**
```css
.parent-a { position: relative; z-index: 1; }
.child-a  { position: relative; z-index: 999; }
.parent-b { position: relative; z-index: 2; }
.child-b  { position: relative; z-index: 1; }
```
`.child-b` renders on top despite `.child-a`'s much higher z-index — `.child-a`'s `999` is only compared *within* `.parent-a`'s context, and `.parent-a` (`1`) loses to `.parent-b` (`2`) at the outer level. `.child-a` can never escape its parent's stacking context regardless of its own z-index value.

**Key principle:** "If some element is in a stacking context that is under some other stacking context, there is no z-index value possible that will bring it on top." The bug lives one level up from where it appears to be.

**Practical takeaway:** massive `z-index` values (`99999`) scattered through a codebase usually signal someone fighting stacking contexts they don't understand. Production codebases typically use a small, deliberate z-index scale via design tokens (`--z-dropdown: 10; --z-modal: 100; --z-toast: 1000;`) — stacking context boundaries, not raw z-index size, determine most real-world stacking behavior.

## Debugging stacking contexts — real tools (verified current via search)

Manually tracing stacking contexts by reading code doesn't scale as component trees grow. Real tools exist:

**"CSS Stacking Context Inspector"** (Chrome and Firefox) — adds a "Stacking Contexts" panel to DevTools showing the full nesting hierarchy as an expandable tree, plus an Elements-panel sidebar showing: an element's parent stacking context, sibling sub-contexts, and — if it creates a new context — exactly which property triggered it.

**`z-context`** (Chrome extension, actively maintained) — similar functionality: shows whether the selected element creates a stacking context and why, its parent context, and its z-index value. Recent updates cover Shadow DOM and `position: sticky` support.

**Microsoft Edge** — has a built-in 3D stacking-context view natively, no extension needed.

**Practical recommendation:** install "CSS Stacking Context Inspector" now — same category of tool as an accessibility linter: not needed constantly, but fast to reach for the moment a z-index bug doesn't make sense from reading code alone.

## Native anchor positioning — cutting-edge, worth knowing exists

```css
.tooltip {
  position-anchor: --my-button;
  position: absolute;
  top: anchor(bottom);
  left: anchor(left);
}
.trigger-button { anchor-name: --my-button; }
```
Positions an element relative to a completely different, non-ancestor element, natively — historically required JS libraries (Popper.js, Floating UI) to calculate coordinates on scroll/resize. Very recent, Chrome-first browser support — know it exists and what problem it solves, not yet reliable for broad production use.

## Where we left off

First slice of Section 5 (Layout Systems) covered: normal flow, block/inline/inline-block, the `display` property (including `display: contents` vs Fragment/Vue multi-root — a genuinely valuable distinction), full positioning system, and a deep dive into z-index/stacking contexts including real debugging tooling. No exercise built yet for this section.

**Remaining Section 5 slices (planned for follow-up sessions):** Flexbox (container + item properties, common patterns), Grid (`grid-template-columns`/`rows`/`areas`, `repeat()`/`minmax()`/`fr`, subgrid, implicit vs explicit grid), and multi-column layout.
