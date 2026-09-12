# Session Summary: Transitions, Animations & Transforms (Section 9)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `transition` basics

```css
.btn {
  transition: background-color 0.3s ease-in-out, transform 0.2s ease;
}
```
**Not every property can transition** — only properties with a defined interpolatable in-between value (`width`, `color`, `opacity`, `transform`). `display` (block/none has no in-between) cannot transition at all — common source of "why isn't my transition working."

**`transition-timing-function` curves:**
```css
ease;       /* default — slow start, fast middle, slow end */
linear;     /* constant speed — feels robotic, rarely wanted */
ease-in;    /* slow start, accelerates — good for elements LEAVING */
ease-out;   /* fast start, decelerates — good for elements ENTERING */
```
**Rule of thumb:** entering elements → `ease-out` (arrive fast, settle gently); exiting elements → `ease-in` (leave gradually, then speed away) — this asymmetry is how polished motion design systems (Material Design, iOS) are actually tuned, not arbitrary.

## `@keyframes` and `animation`

```css
@keyframes slide-in {
  from { transform: translateX(-100%); opacity: 0; }
  to { transform: translateX(0); opacity: 1; }
}
.el {
  animation: slide-in 0.5s ease-out forwards;
}
```

**`animation-fill-mode: forwards` — the property people forget:** without it, once an animation finishes, the element **snaps back to its original pre-animation CSS state** — animations don't persist their final keyframe by default. Forgetting this causes an element to visibly flash back to invisible/off-screen the instant an animation completes.

**`transition` vs `animation` — when to use which:** `transition` for simple state changes triggered by something (hover, class toggle) — implicit current-to-new, one time. `animation` for multi-step sequences, auto-run on load, looping, or precise intermediate-state control — more powerful, more verbose.

## `transform` functions

```css
transform: translate(20px, 10px);
transform: rotate(45deg);
transform: scale(1.5);
transform: skew(10deg, 5deg);
transform: perspective(500px) rotateY(45deg); /* 3D */
```

**Critical performance reasoning:** `transform` and `opacity` are the two properties the browser's compositor can animate almost for free, without triggering layout recalculation or repaint of surrounding content. This is why "animate `transform`/`opacity`, avoid `width`/`height`/`top`/`left`" is standard practice — animating `left` forces layout recalculation every frame (expensive); `transform: translateX()` achieves the same visual movement touching only the compositor layer (cheap). A commonly tested "do you understand CSS performance" interview question.

## Individual transform properties — recent

```css
/* OLD — must combine everything into one property string */
transform: translateX(20px) rotate(45deg) scale(1.2);

/* NEW — independent properties */
translate: 20px 0;
rotate: 45deg;
scale: 1.2;
```
**Why it matters:** previously, animating rotation via CSS while JS independently controls position required manually concatenating the whole transform string on every change — an annoying coordination problem. Individual properties can be set/transitioned/animated completely independently without overwriting each other — genuinely useful when combining CSS-driven and JS-driven transforms on the same element.

## `transform-origin` and `perspective`

```css
.el { transform: rotate(45deg); transform-origin: top left; } /* pivot point, not center */
```
```css
.container { perspective: 500px; }     /* on the PARENT — establishes 3D viewing distance */
.child { transform: rotateY(45deg); }  /* viewed through parent's perspective */
```
`perspective` must be on the parent (or via `perspective()` function on the element itself) for 3D transforms to have real depth — without it, `rotateY`/`rotateX` visually flatten/distort instead of looking like genuine 3D rotation.

## Scroll-driven animations — flagged recent gap area

```css
.reveal {
  animation: fade-in-on-scroll linear;
  animation-timeline: view();
  animation-range: entry 0% cover 40%;
}
```

**Paradigm shift:** traditionally, animating an element as it scrolls into view required JS (`IntersectionObserver` + class toggling, or manual scroll-event calculations, often with performance issues). **`animation-timeline` lets the browser drive animation progress directly from scroll position, natively, with zero JS.**

**Two timeline sources:**
- `animation-timeline: view()` — progress tied to how far the element itself has scrolled through the viewport. Use for "animate this element in as it enters view" (most common pattern).
- `animation-timeline: scroll()` — progress tied to the scroll container's own scroll position. Use for things like a progress bar filling based on overall page scroll.

**Current status:** solid Chrome/Edge support, more limited elsewhere — use `@supports` fallback or treat as progressive enhancement (unsupported browsers see the final/static state, not a broken page) rather than relying on it for critical functionality.

## View Transitions API — cutting-edge

```js
document.startViewTransition(() => {
  updatePageContent();
});
```
```css
::view-transition-old(root), ::view-transition-new(root) { animation-duration: 0.5s; }
```
**Problem solved:** smooth, native cross-fade/morph transitions between two DOM states (or between SPA page navigations in supporting frameworks) — previously required manual animation choreography or heavy libraries. Browser captures "before"/"after" snapshots and cross-fades between them. Genuinely bleeding-edge — know the concept, not yet reliable for broad production use.

## `will-change` — double-edged performance hint

```css
.el { will-change: transform; }
```
Tells the browser to promote the element to its own compositor layer in advance, avoiding a layer-promotion delay when an animation starts.

**Senior-level warning:** not free — compositor layers consume GPU memory. Applying it broadly/permanently (every card in a long list, or leaving it on indefinitely) can *hurt* performance via excessive memory pressure from too many layers.

**Correct usage:** apply briefly, right before an animation starts (often via JS — add on `mouseenter`, remove on `mouseleave`/animation end) — not as a permanent blanket optimization on everything that might someday animate.

## Where we left off

Section 9 (Transitions, Animations & Transforms) covered in full, at senior-frontend depth throughout. The `transform`/`opacity` compositor-performance story and scroll-driven animations flagged as the two most currently load-bearing, interview-relevant concepts. No exercise built for this section.

Next up per the CSS reference doc: **Section 10 — Modern Layout & Architecture Features** (native CSS nesting with `&` syntax, `@scope`, logical properties recap, `aspect-ratio`, `gap` in flexbox) — another section with flagged recent-gap topics (native nesting, `@scope`).
