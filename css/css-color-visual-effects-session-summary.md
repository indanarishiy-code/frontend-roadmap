# Session Summary: Color & Visual Effects (Section 8)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `background` — sub-properties

```css
.el {
  background-image: url("hero.jpg");
  background-position: center top;
  background-size: cover;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-clip: padding-box;
  background-color: #333; /* shown while image loads or if it fails */
}
```

**`background-size: cover` vs `contain`:**
- `cover` — fills the entire element, crops overflow. Use when the image should completely fill the space.
- `contain` — fits entirely within the element without cropping, may letterbox. Use when the entire image must stay visible (product photos, logos).

**`background-attachment: fixed`** — classic parallax effect (background stays fixed while content scrolls). **Real caveat:** historically janky scroll performance on mobile — some mobile browsers don't support it well or disable it. Test on actual devices.

**`background-clip: text`** — popular modern technique for gradient text:
```css
.gradient-text {
  background: linear-gradient(90deg, blue, purple);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
}
```
Clips the background to the shape of the text; `color: transparent` lets the gradient show through the letters.

## Gradients

```css
background: linear-gradient(90deg, red, blue);
background: radial-gradient(circle, red, blue);
background: conic-gradient(from 0deg, red, yellow, green, blue, red);
```
**`conic-gradient`** — least commonly known; sweeps colors around a center point like a clock face rather than linearly/radially. Useful for pie-chart visuals or color-wheel pickers in pure CSS.

**Connection to Section 1:** gradients can use `oklch()` color stops — avoids the muddy grey/brown transition zones that `rgb()`/`hsl()` gradients often produce, per oklch's perceptual uniformity advantage.

## Shadows and filters

```css
box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.2); /* inset — shadow INSIDE the element */
box-shadow: 0 0 0 3px blue, 0 0 0 6px white;     /* multiple shadows — layered borders/focus rings technique */

filter: blur(4px);
filter: drop-shadow(0 4px 6px rgba(0,0,0,0.3));

backdrop-filter: blur(10px); /* blurs what's BEHIND the element */
```

**`box-shadow` vs `filter: drop-shadow()` — important distinction:** `box-shadow` always follows the element's rectangular (or rounded) box shape regardless of visible content. `filter: drop-shadow()` follows the actual visible pixel/alpha shape — critical for a PNG icon with transparency or an irregular SVG, where `box-shadow` would shadow the invisible bounding rectangle instead of the visible shape.

**`backdrop-filter`** — applies the effect to whatever is BEHIND the element through its transparency, not to the element's own content. This is the "frosted glass" UI effect technique (macOS control center, many modern modal overlays).

## `opacity` vs alpha channel — real practical distinction

```css
.a { opacity: 0.5; }                    /* fades the ENTIRE element — text, borders, children, everything */
.b { background: rgba(0, 0, 0, 0.5); }  /* only background is translucent; text/children stay fully opaque */
```
**Common mistake:** using `opacity` to make a card's background translucent also fades its text/borders/nested images uniformly. Use alpha-channel color (`rgba()`, `hsl(... / 0.5)`, 8-digit hex) on the specific property instead, when only the background should be translucent.

## `border-radius`

```css
border-radius: 8px;                                    /* all corners */
border-radius: 8px 16px 8px 16px;                       /* TL, TR, BR, BL */
border-radius: 50%;                                     /* circle/ellipse on a square/rect */
border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%;       /* elliptical per-corner — "blob" shapes */
```
The elliptical syntax (`horizontal-radii / vertical-radii`) lets each corner have independent horizontal and vertical curves — how organic "blob" background shapes (popular in modern hero sections) are built in pure CSS.

## `clip-path`

```css
clip-path: circle(50%);
clip-path: polygon(50% 0%, 0% 100%, 100% 100%); /* triangle */
clip-path: inset(10px 20px 10px 20px round 8px);
```
Clips to a shape — anything outside is genuinely not painted (real performance implications for complex layered UIs, not just visually hidden). Useful for non-rectangular crops, custom-shaped buttons, or reveal animations (animatable).

## Blend modes — visual comparison demonstrated

```css
mix-blend-mode: multiply;    /* darkens overlap — like stacking semi-transparent films */
mix-blend-mode: screen;      /* lightens overlap — works well on dark backgrounds for glow effects */
mix-blend-mode: difference;  /* inverts based on color difference — keeps overlay text readable regardless of background */
background-blend-mode: overlay; /* blends multiple BACKGROUND LAYERS within the same element */
```

**`mix-blend-mode`** blends an element with whatever's behind it (other elements/page background). **`background-blend-mode`** blends multiple background layers within the same single element (e.g. a gradient over a solid background-color on one box) against each other — a different scope than `mix-blend-mode`.

**`difference` — the genuinely practical one:** demonstrated keeping text legible over an unpredictable multi-color gradient background by inverting based on color difference at each point. Real technique for overlay text on dynamic/unpredictable backgrounds (video players, image carousels) where guaranteed contrast isn't otherwise possible.

**Overall assessment:** blend modes are mostly niche/creative-use territory — more common in illustrative/marketing design than typical dashboard/CMS UI — but worth recognizing when encountered, and `difference` specifically has genuine practical accessibility/legibility value.

## Where we left off

Section 8 (Color & Visual Effects) covered in full, with a visual widget demonstrating `multiply`, `screen`, `difference`, and `background-blend-mode: overlay` side by side. The `opacity` vs alpha-channel distinction and `box-shadow` vs `filter: drop-shadow()` flagged as the two most practically important, easy-to-misuse concepts in this section.

Next up per the CSS reference doc: **Section 9 — Transitions, Animations & Transforms** (`transition`, `@keyframes`/`animation`, `transform` functions, individual transform properties, `transform-origin`/`perspective`, scroll-driven animations — a flagged recent-gap topic — View Transitions API, `will-change`).
