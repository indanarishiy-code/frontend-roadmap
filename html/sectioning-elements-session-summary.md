# Session Summary: Sectioning Elements (`header`, `footer`, `main`, `nav`, `article`, `section`, `aside`)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<main>`
- Primary, unique content of the page — excludes repeated site chrome (nav, header, footer, sidebars)
- Exactly one per page; must not be nested inside `<article>`, `<aside>`, `<header>`, `<footer>`, or another `<main>`
- Real accessibility value: screen reader users can jump straight to it via a landmark shortcut

## `<header>`
- Introductory content (logo, title, intro)
- Scoped to whatever it's inside — not always the page masthead; can appear multiple times (page-level header, plus one inside each `<article>` for title/author/date)

## `<footer>`
- Closing/supplementary content (copyright, related links, author bio)
- Also scopable — once per page, and again inside individual `<article>` elements

## `<nav>`
- Wraps a block of *major* navigation links — not every group of links, just primary navigation (main site nav, pagination, table of contents)
- A page can have multiple `<nav>` elements; label each via `aria-label` so screen readers can distinguish them:
```html
<nav aria-label="Main">...</nav>
<nav aria-label="Breadcrumb">...</nav>
```

## `<article>` vs `<section>` vs `<div>` — the core decision logic

**Step 1:** Is this purely a layout/styling wrapper with no independent meaning? → `<div>`

**Step 2 (if it does carry meaning):** Would this content still make complete sense if pulled out and displayed somewhere else entirely, independent of this page?
- Yes → `<article>` (self-contained, could be syndicated/reused — blog post, news story, forum comment, product card)
- No, only makes sense within this page's context → `<section>` (thematic grouping, generally with a heading, doesn't need to stand alone)

```html
<!-- ARTICLE: standalone, could be syndicated -->
<article>
  <h2>How to Learn CSS Grid</h2>
  <p>Full blog post content...</p>
</article>

<!-- SECTION: only makes sense as part of THIS page -->
<section>
  <h2>Table of Contents</h2>
  <ul>...</ul>
</section>

<!-- DIV: pure layout, no semantic meaning -->
<div class="two-column-layout">
  <div class="sidebar">...</div>
  <div class="content">...</div>
</div>
```

**Nesting is fine both ways:** an `<article>` can contain multiple `<section>`s (e.g. a product review split into "Pros"/"Cons" sections), and a `<section>` can contain an `<article>` (e.g. a "Featured Post" section on a homepage).

### Most common real mistake
Using `<section>` as a `<div>` replacement just because it "sounds more semantic" — even for plain layout wrappers with no heading. This actively hurts accessibility: every `<section>` with an accessible name (heading or `aria-label`) gets exposed as a "region" landmark to screen readers. Unnamed/heading-less sections either clutter landmark navigation meaninglessly or get flagged by a11y linters.

**Practical rule:** if you wouldn't naturally put a heading at the top of it, it's probably a `<div>`, not a `<section>`.

## `<aside>`
- Content tangentially related to surrounding content — sidebars, pull quotes, ads, "related articles"
- Not essential to understanding the main content, but relevant enough to include nearby

## Applying this to real Vue/framework component habits

**Key insight:** a component's *file name* (e.g. `ProfileSection.vue`) is purely a codebase organization convention — it has no obligation to match the actual root HTML tag rendered. Naming something "...Section" and rendering a `<div>` root is completely normal and fine.

**When the root SHOULD actually be `<section>`:** apply the same test at the component level — does this chunk of the page represent a genuine thematic region with its own heading, that a screen reader user would meaningfully benefit from jumping to as a landmark? For a detail page split into components like `ProfileSection`, `ReviewsSection`, `SpecsSection` — if each has its own heading and represents a genuinely distinct part of the page's content, `<section>` is the semantically correct root, not just an acceptable one. Using `<div>` there works but leaves real accessibility value (landmark navigation) on the table.

**When `<div>` is still correct regardless of component name:** if the component's root is purely a styling/layout wrapper (flex/grid container) with no heading and no thematic identity — no region to expose.

**Practical forward-looking habit:** no need to refactor existing code, but going forward, swap `<div>` root to `<section>` when a component genuinely represents "a distinct heading-led chunk of this page's content" — low-cost, real accessibility improvement.

## Where we left off

`<header>`/`<footer>`/`<main>`/`<nav>`/`<article>`/`<section>`/`<aside>` slice closed, with the `<article>` vs `<section>` vs `<div>` distinction directly connected to real Vue component habits. Remaining Section 2 pieces: `<div>` and when it's correct (partially covered above, could revisit standalone), heading elements (`<h1>`–`<h6>`, `<hgroup>`), and the outline algorithm (historical vs current spec status).
