# Session Summary: Cascade, Specificity & Inheritance (Section 3)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: `@layer` was covered in depth in the previous session (Section 1) and is referenced here as part of the same cascade hierarchy.*

## The cascade — full resolution order

When multiple rules target the same element/property, resolution follows this exact sequence:

1. **Origin and importance** — user-agent styles < user styles < author styles < author `!important` < user `!important` < user-agent `!important` (note: `!important` actually flips priority order between origins)
2. **Layers** (`@layer`) — later-declared layer wins regardless of specificity, but only within the same importance tier
3. **Specificity** — higher specificity wins
4. **Source order** — if everything above ties exactly, the later rule in the stylesheet wins

**Senior-level insight:** specificity is only step 3 of 4 — layers override it entirely (step 2), and origin/`!important` overrides layers (step 1). This is exactly why `@layer` is so powerful: it operates at a higher priority tier than specificity, not as a specificity trick.

## Specificity calculation

Tuple: `(inline, IDs, classes/attributes/pseudo-classes, elements)`
```css
p { }                         /* (0,0,0,1) */
.card { }                     /* (0,0,1,0) */
#header { }                   /* (0,1,0,0) */
#header .card.active p { }    /* (0,1,2,1) */
<p style="color: red">        /* (1,0,0,0) — inline always wins over any selector */
```

**Critical rule:** compared column by column, left to right — never added as a single number. `(0,1,0,0)` (one ID) beats `(0,0,99,99)` (99 classes + 99 elements) — no amount of low-specificity selectors can out-compete one higher-tier selector.

**Contribute zero to specificity:** universal selector, combinators themselves, `:where()`:
```css
* { }                /* (0,0,0,0) */
div > p { }          /* (0,0,0,2) — combinator adds nothing, just counts elements */
:where(.card) p { }  /* (0,0,0,1) — :where() contributes 0, only 'p' counts */
```

## Inheritance

**Inherited by default** (mostly text/typography): `color`, `font-family`, `font-size`, `line-height`, `text-align`, `visibility`, `cursor`, `list-style`.

**Not inherited by default** (mostly box/layout): `margin`, `padding`, `border`, `width`, `height`, `background`, `display`, `position`.

**Reasoning behind the split:** typography inherits because consistent text styling flowing down a document is almost always wanted (set `font-family` once on `body`). Box/layout properties don't inherit because inheriting `border`/`margin` would be actively harmful — every nested `<div>` would get the ancestor's border/margin, essentially never desired.

## `inherit` / `initial` / `unset` / `revert`

```css
.child {
  color: inherit;   /* force-take parent's computed value, even for non-inheriting properties */
  border: initial;  /* reset to spec-defined default, ignoring inheritance/cascade entirely */
  margin: unset;    /* 'inherit' if property normally inherits, 'initial' if it doesn't */
  color: revert;    /* reset to what browser's default UA stylesheet would set — undoes AUTHOR-level styles specifically */
}
```

**`revert` — genuinely underused, senior-level:** useful for "undoing" your own or a library's/reset's styling to fall back to native browser appearance — e.g. `all: revert` on a `<button>` to strip an aggressive CSS reset's button normalization.

**Key clarification: `revert` doesn't lock the property afterward** — it's just another cascade value, like `red` or `10px`. Declarations after it (same rule or later/more specific rules) apply completely normally on top:
```css
button {
  all: revert;           /* undo reset, get native button appearance back */
  background: #3b82f6;   /* then apply own styling on top, normally */
  color: white;
}
```
Per-property revert works the same way — reverting `border` doesn't affect `background` or other properties in the same rule at all.

## CSS custom properties (variables)

```css
:root {
  --primary-color: #3b82f6;
  --spacing-unit: 8px;
}
.btn {
  background: var(--primary-color);
  padding: calc(var(--spacing-unit) * 2);
}
```

**Senior-level nuances:**
1. **Custom properties DO inherit and cascade normally** — unlike some CSS mechanisms, they follow standard inheritance rules and can be reassigned anywhere in the tree to affect everything below:
```css
.dark-theme { --primary-color: #60a5fa; } /* overrides for this subtree */
```
2. **Fallback values:** `color: var(--undefined-var, blue);` — uses `blue` if `--undefined-var` isn't set.
3. **Invalid values become "guaranteed-invalid," not silently ignored:** if a custom property's value is invalid for its usage context (e.g. `--size: not-a-number;` used in `width: var(--size);`), the property falls back to its **initial value** — not to whatever was previously cascaded. A real gotcha if unaware of this specific behavior.

## `@property` — typed custom properties

```css
@property --progress {
  syntax: '<percentage>';
  inherits: false;
  initial-value: 0%;
}
```

**Why it exists:** plain custom properties are untyped strings to the browser — they cannot be animated/transitioned, since the browser can't interpolate between arbitrary string values. `@property` gives a custom property an actual type (`<percentage>`, `<color>`, `<length>`, etc.), unlocking genuine CSS-only animation:
```css
.progress-bar { transition: --progress 0.3s ease; }
.progress-bar:hover { --progress: 100%; }
```
Without `@property`, this silently does nothing. Genuinely current capability — often the missing piece when a custom property won't transition/animate as expected.

## Where we left off

Section 3 (Cascade, Specificity & Inheritance) covered in full, directly connecting back to `@layer` from the previous session as part of the same cascade resolution hierarchy. Includes a clarifying exchange on how `revert` participates in the cascade rather than locking a property. No exercise built for this section.

Next up per the CSS reference doc: **Section 4 — Box Model** (content/padding/border/margin, `box-sizing`, margin collapsing, `overflow`, `outline` vs `border`).
