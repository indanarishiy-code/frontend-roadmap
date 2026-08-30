# Session Summary: Performance-Relevant HTML — Critical Rendering Path (Section 14)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: `loading="lazy"`/`fetchpriority` (Embedded Content), resource hints `preload`/`prefetch`/`preconnect` (`<head>` contents/Metadata sessions), and `async`/`defer` on `<script>` (`<head>` contents session) were already covered in depth.*

## The critical rendering path — core sequence

```
HTML bytes → DOM tree (parsing)
CSS bytes  → CSSOM tree
DOM + CSSOM → Render Tree
Render Tree → Layout (compute positions/sizes)
Layout → Paint (draw pixels)
```

## Why CSS is render-blocking by default

```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```
The browser won't paint anything to screen until the CSSOM is fully built (all CSS parsed) — it needs to know styling before computing a render tree.

**The deeper "why" (not just a technical limitation):**
- Browsers *could* render unstyled HTML immediately and re-render once CSS arrives — some old browsers did this
- This created **FOUC (Flash of Unstyled Content)** — a visibly jarring flash from ugly/unstyled to styled, sometimes with elements visibly jumping as layout recalculates
- This was disorienting enough that browser vendors converged on: hide rendering entirely until styling is settled, rather than show something ugly first

**Why CSS and JS aren't symmetrical here:** JS's job is usually interactivity — its absence at first paint is invisible to the user (page just isn't interactive yet, often fine momentarily). CSS's job is the visual appearance itself — its absence at first paint is visually obvious and jarring. This is why deferring JS is safe/encouraged but deferring CSS isn't treated the same way.

**Important nuance — it's not "block everything":** HTML can still be parsed and even partially built into the DOM in the background while CSS downloads (consistent with the parsing/tokenization mechanics covered earlier). What's specifically blocked is **first paint** (pixels appearing on screen), not the entire parsing pipeline.

## Why JavaScript blocks HTML parsing (recap, ties back to `<head>` session)

```html
<script src="app.js"></script>   <!-- no async/defer -->
```
Plain `<script>` tags pause HTML parsing entirely while downloading/executing, because the script might use `document.write()` or otherwise modify the DOM in ways that would invalidate parsing if it continued in parallel. This is exactly why `defer`/`async`/`type="module"` exist.

## Combined practical picture

```html
<head>
  <link rel="stylesheet" href="styles.css">   <!-- blocks rendering -->
  <script src="app.js"></script>               <!-- blocks parsing -->
</head>
```
Unoptimized sequence: parse HTML → hit CSS link → download+parse CSS (rendering blocked) → hit script tag → download+execute JS (parsing blocked) → continue parsing rest of HTML → finally render.

**Practical guidance this explains:** CSS in `<head>` (accept the render-block, since correct styles before first paint is the goal anyway), scripts with `defer` or at the end of `<body>` (avoid blocking parsing unnecessarily).

## Critical CSS — the modern technique addressing this tradeoff directly

Extract just the CSS needed for above-the-fold content and inline it directly in `<head>` as a `<style>` block; load the rest of the CSS asynchronously. Gets fast, correct-looking first paint (tiny inlined critical CSS loads instantly) without waiting for the entire stylesheet. Tools like the `critical` npm package automate this extraction. A genuinely real, employed performance technique in production.

## Why this matters for interviews

"Explain the critical rendering path" is a common frontend interview question specifically because it tests whether the "CSS in head, scripts at bottom/deferred" advice is understood as a mechanism, not just memorized as a rule.

## Where we left off

Section 14 (Performance-Relevant HTML) covered — the one genuinely new piece being critical rendering path fundamentals, including a real back-and-forth on *why* CSS is render-blocking by default (FOUC prevention, the CSS/JS asymmetry) and the critical-CSS technique as the modern answer to that tradeoff. Everything else in this section (`loading="lazy"`, resource hints, `async`/`defer`) was already covered in earlier sessions.

Next up per the HTML reference doc: **Section 15 — Deprecated/Historical** (`<center>`, `<font>`, `<marquee>`, `<blink>`, frame-based layouts, presentational attributes like `bgcolor`/`align` and why they were replaced by CSS) — a lighter, mostly recognition-level section.
