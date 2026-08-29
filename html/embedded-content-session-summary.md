# Session Summary: Embedded Content (Section 5)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<img>` — modern attribute set

```html
<img src="cat.jpg" alt="A cat sleeping on a windowsill"
     srcset="cat-480.jpg 480w, cat-800.jpg 800w, cat-1200.jpg 1200w"
     sizes="(max-width: 600px) 100vw, 50vw"
     loading="lazy" decoding="async" fetchpriority="high">
```

- **`alt`** — required. Describes image for screen readers/broken images/SEO. Empty `alt=""` is valid and correct for purely decorative images (tells screen readers to skip it) — never omit `alt` entirely.
- **`srcset` + `sizes`** — responsive images. `srcset` lists candidate files with actual widths (`480w`); `sizes` tells the browser how wide the image displays at different viewport widths. Browser auto-picks the best-fit file — no JS or duplicated media queries needed.
- **`loading="lazy"`** — defers loading offscreen images until scrolled near; improves initial load. Don't use on above-the-fold images — defeats the purpose.
- **`decoding="async"`** — lets browser decode off the main thread, avoiding a rendering stall.
- **`fetchpriority="high"|"low"`** — hints which images matter most (e.g. hero image) vs which can wait.

## `<picture>` / `<source>`

For serving genuinely different image files based on conditions (not just resolution variants — that's `srcset`'s job):
```html
<picture>
  <source srcset="photo.avif" type="image/avif">
  <source srcset="photo.webp" type="image/webp">
  <img src="photo.jpg" alt="Description">
</picture>
```
Browser picks the first supported `<source>`, falls back to `<img>`. Common uses: modern format with fallback (AVIF/WebP → JPEG), or art direction (different crop/composition for mobile vs desktop).

## `<audio>` / `<video>` / `<track>`

```html
<video controls poster="thumbnail.jpg" preload="metadata">
  <source src="movie.webm" type="video/webm">
  <source src="movie.mp4" type="video/mp4">
  <track kind="captions" src="captions.vtt" srclang="en" label="English">
</video>
```
- Multiple `<source>`s for format fallback, same pattern as `<picture>`
- `<track kind="captions">` — genuinely important for accessibility, not optional polish
- `controls` shows native UI; omit if building custom controls via JS

## `<iframe>` — security-relevant attributes

```html
<iframe src="https://trusted-embed.com/widget"
        sandbox="allow-scripts allow-same-origin"
        loading="lazy"
        referrerpolicy="no-referrer">
</iframe>
```
- **`sandbox`** — restricts what the embedded page can do; no value = blocks everything by default (scripts, forms, popups). Opt back in per-capability (`allow-scripts`, `allow-forms`, `allow-popups`, etc.) — a real security boundary for embedding untrusted third-party content.
- **`allow`** — grants specific browser features (camera, microphone, fullscreen) via Permissions Policy syntax
- **`referrerpolicy`** — controls how much referrer info leaks to the embedded page

## `<embed>` / `<object>` / `<param>`

Legacy-leaning elements for embedding external resources (PDFs, historically Flash, plugins):
```html
<object data="document.pdf" type="application/pdf">
  <p>PDF cannot be displayed. <a href="document.pdf">Download it</a>.</p>
</object>
```
Rare in modern frontend work — mostly superseded by `<video>`/`<audio>`/`<iframe>` or JS-based viewers. Worth recognizing in legacy code, not something to reach for in new projects.

## `<map>` / `<area>` — image maps

```html
<img src="diagram.png" usemap="#regions" alt="Diagram">
<map name="regions">
  <area shape="rect" coords="0,0,100,100" href="part1.html" alt="Part 1">
</map>
```
Defines clickable regions on a single image. Niche today — mostly superseded by CSS-positioned elements over an image, or SVG with clickable shapes.

## `<canvas>`

```html
<canvas id="chart" width="400" height="200"></canvas>
```
Blank drawing surface — content drawn imperatively via JS (`getContext("2d")` or WebGL), not declarative HTML. Used for games, custom charts, image manipulation. **Important:** nothing inside is in the accessibility tree by default — accessible fallback content and ARIA labeling must be added deliberately if the canvas conveys real information.

## SVG embedding — three approaches, real tradeoffs

- **Inline `<svg>...</svg>`** — fully stylable/scriptable via CSS/JS; each instance is separate DOM (can bloat markup if reused often)
- **`<img src="icon.svg">`** — simple, cacheable, but can't be styled/manipulated via CSS/JS from the parent page
- **`<object data="icon.svg">`** — retains some scriptability, rarely used over the other two today

## Practical priority for this role/market

Given target markets and modern frontend work, the genuinely high-value items to hold onto solidly:
- `<img>`'s `srcset`/`sizes`/`loading` — performance, comes up constantly in real work and interviews
- `<picture>` for format fallback (AVIF/WebP → JPEG)
- `<iframe sandbox>` for security when embedding third-party content

Lower priority / know-it-exists level: `<embed>`/`<object>`/`<param>`, `<map>`/`<area>` (legacy/niche).

## Where we left off

Section 5 (Embedded Content) covered in full pass: `<img>` modern attributes, `<picture>`/`<source>`, `<audio>`/`<video>`/`<track>`, `<iframe>` security attributes, `<embed>`/`<object>`/`<param>`, `<map>`/`<area>`, `<canvas>` (including the accessibility-tree gap), and SVG embedding tradeoffs. No exercise done for this section yet — could revisit responsive images (`srcset`/`sizes`) hands-on later if useful.

Next up per the HTML reference doc: **Section 6 — Tables** (`<table>`, `<caption>`, `<colgroup>`/`<col>`, `<thead>`/`<tbody>`/`<tfoot>`, `<tr>`/`<th>`/`<td>`, `scope`/`headers`/`id` for accessible tables, `rowspan`/`colspan`).
