# Session Summary: Selectors (Section 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Basic selectors

```css
p { }           /* type */
.card { }       /* class */
#header { }     /* ID */
* { }            /* universal */
```
**Senior nuance:** most modern CSS architecture (BEM, utility-first, CSS Modules) deliberately avoids ID selectors for styling — not because they don't work, but their high specificity (Section 3) makes them disproportionately hard to override later. Seeing ID selectors in production CSS is often a review red flag.

## Attribute selectors

```css
[disabled] { }
[type="text"] { }
[class^="btn-"] { }   /* starts with */
[class$="-icon"] { }  /* ends with */
[class*="active"] { } /* contains anywhere */
[lang|="en"] { }      /* exact match OR starts with "en-" */
```

**When conditional classes (the more idiomatic Vue/React default) are the right call:** anything driven by your own component's internal state you already control — no reason to reach for attribute selectors instead.

**When attribute selectors genuinely win:**
1. **Native HTML attributes not mirrored in JS classes** — `input:required`, `a[target="_blank"]`, `input[type="email"]:invalid` — the browser already manages these natively (ties to Constraint Validation API); no need to duplicate that state into a class.
2. **Styling markup you don't control** — CMS output, third-party widgets, user-generated/rendered content — `[class*="wp-image-"]` etc. is the strongest case for `^=`/`$=`/`*=`.
3. **`data-*` as a deliberate state API, separate from `class`** — some design systems (Radix, shadcn/ui) use `data-*` for state (open/closed, loading, selected) and reserve `class` purely for styling identity:
```html
<div data-state="loading" data-variant="primary">
```
```css
[data-state="loading"] { opacity: 0.5; }
```
4. **Matching attribute *values* with no class equivalent** — e.g. `[href^="mailto:"]::before { content: "✉️ "; }` — no reasonable way to "just add a class" based on detecting a URL scheme.

**Honest takeaway:** conditional classes are correct most of the time for component-internal state; attribute selectors earn their place for native browser-managed state, uncontrolled markup, or deliberate state/style separation architecture.

## Combinators

```css
div p { }        /* descendant — any p anywhere inside div */
div > p { }      /* child — only direct children */
div + p { }      /* adjacent sibling */
div ~ p { }      /* general sibling */
```
**Senior distinction:** `>` (child) generally preferred over plain descendant in component-based architecture — descendant selectors can accidentally match nested instances of the same component (a card inside a card); `>` scopes precisely to intended direct structure.

## Pseudo-classes — key distinctions

```css
:focus            /* fires for both mouse clicks and keyboard nav */
:focus-visible    /* only for likely keyboard/assistive-tech navigation */
:focus-within     /* matches a PARENT if any descendant has focus */
```
**`:focus` vs `:focus-visible`:** the modern, correct way to style focus indicators without the old "just remove outline" anti-pattern — avoids the visual noise of a focus ring on every mouse click while still showing it for keyboard navigation (connects to Section 14, accessibility-relevant CSS).

**`:focus-within`** — style a parent container when any input inside is focused, no JS:
```css
.form-group:focus-within { border-color: blue; }
```

Other pseudo-classes covered: `:hover`, `:active`, `:first-child`/`:last-child`, `:nth-child()`/`:nth-of-type()`, `:not()`, `:is()`, `:where()` (zero specificity, covered previously), `:empty`, `:checked`, `:valid`/`:invalid`, `:target`.

## Pseudo-elements

```css
::before / ::after     /* generated content, requires content: "" */
::first-line
::first-letter
::selection             /* user-selected/highlighted text */
::placeholder
::backdrop               /* dialog/popover backdrop, covered earlier */
::marker                 /* list bullet/number itself */
```

## Grouping selectors

```css
h1, h2, h3 { color: navy; }
```
Comma-separated, applies identically to each.

## `:has()` — the "parent selector" CSS never had (deep dive)

### What it is
For CSS's entire history, selectors could only match downward/rightward (element + descendants). `:has()` breaks that:
```css
.card:has(img) { padding: 0; }
.form-group:has(input:invalid) { border-color: red; }
```
Reads as "select `.card`, but only if it has a descendant matching `img`" — a genuine paradigm shift, not just a convenience.

### Real value: eliminates a category of JS
```css
/* Style a parent based on a checked child, no JS needed */
.list-item:has(input:checked) { background: lightblue; }
```
Replaces the old pattern of JS watching child state and toggling a parent class manually.

### Combinators work inside `:has()` — real new expressive power
```css
section:has(> h2:first-child) { }      /* has an h2 as direct first child */
.card:not(:has(img)) { }                /* select based on ABSENCE of a descendant — impossible before :has() */
h2:has(+ p) { margin-bottom: 0.5rem; }  /* select based on what comes AFTER it — a decade-old CSS limitation, solved */
```

### Real limitations/gotchas
1. **Specificity:** calculated like `:is()` — contributes the specificity of its most specific argument, NOT zero like `:where()`.
2. **Performance (real, not theoretical):** requires the browser to look forward/downward in the DOM to evaluate a rule on an ancestor — architecturally more expensive than top-down matching. Modern engines optimize this well, but broad, unscoped use (e.g. `body:has(.rare-class)`) on very large/deep DOM trees can have measurable cost. Nuanced current discussion: not "avoid it," but "don't blanket-replace every JS toggle without considering scale."
3. **Browser support solid since ~2023** across major browsers; needs `@supports` fallback if supporting genuinely old browsers.

### Roadmap connection
Directly maps to the suggested verification exercise (rebuild a component using `:has()` instead of a JS-driven class toggle) — a custom dropdown/select where "open" state is currently a JS-toggled class is a good real candidate, potentially combinable with `:focus-within`.

## Where we left off

Section 2 (Selectors) covered in full, including a genuine deep dive into `:has()` as one of the flagged recent-gap topics, plus a practical discussion on when attribute selectors are worth reaching for versus idiomatic conditional classes. No hands-on exercise built yet — the `:has()`-based dropdown/JS-toggle-replacement exercise from the roadmap doc remains a good candidate for later.

Next up per the CSS reference doc: **Section 3 — The Cascade, Specificity & Inheritance** (how the cascade resolves conflicts, specificity calculation, `@layer` — already touched on — inheritance rules, `inherit`/`initial`/`unset`/`revert`, CSS custom properties, `@property`). This is another flagged recent-gap section.
