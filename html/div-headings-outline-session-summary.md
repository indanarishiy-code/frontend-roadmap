# Session Summary: `<div>`, Headings, `<hgroup>`, Outline Algorithm (closes Section 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<div>` — when it's actually correct

- Correct whenever content is purely structural/layout with no independent thematic meaning and no heading
- Concrete valid uses: JS/CSS hooks with no semantic role (styling class, JS ref wrapper), pure layout grouping (flex/grid containers, spacing wrappers)
- **Not a fallback to feel guilty about** — it's the intentional right choice whenever no other element's semantics apply. The mistake is only ever using `<div>` when something more specific fits, not using `<div>` in general

## Heading elements — `<h1>`–`<h6>`

Define document structural hierarchy, like nested book chapters/sections — not just "big bold text."

```html
<h1>Page Title</h1>
  <h2>Major Section</h2>
    <h3>Subsection</h3>
  <h2>Another Major Section</h2>
```

**Core rule:** never skip heading levels for styling reasons (e.g. `<h1>` straight to `<h4>` because of font size). Screen reader users navigate by heading level as a table-of-contents shortcut — skipping breaks that. Use CSS `font-size` for visual sizing, not the wrong heading level.

**Current best practice:** exactly one `<h1>` per page. The spec technically permits multiple `<h1>`s across different sectioning contexts (via the outline algorithm), but since that algorithm was never implemented by browsers/screen readers, one-`<h1>`-per-page remains the safe, practical rule.

## `<hgroup>`

Groups a heading with associated secondary content (subtitle/tagline) as a single semantic unit, so the secondary text isn't mistaken for its own heading level:

```html
<hgroup>
  <h1>Frontend Roadmap</h1>
  <p>A self-study guide for frontend interviews</p>
</hgroup>
```

The `<p>` is explicitly not a heading — `<hgroup>` signals it's a subtitle. Niche in practice — mainly landing pages/hero sections, not everyday use.

## The outline algorithm — historical, not real behavior

**What it was supposed to do:** originally proposed that sectioning elements (`<section>`, `<article>`, `<aside>`, `<nav>`) would each create their own "outline," letting headings restart numbering relative to their containing section — theoretically allowing `<h1>` inside every `<article>`/`<section>` without it meaning "top-level page heading."

**What actually happened:** no browser ever implemented it, no screen reader ever supported it. Formally removed/marked obsolete in the HTML spec. Some outdated tutorials and interview prep material still reference it as if it's real functioning behavior — it isn't.

**Practical consequence:** headings are read literally by their number regardless of containing sectioning element. This is exactly why "one `<h1>` per page, proper sequential `<h2>`–`<h6>` nesting" is the real, safe rule — not a theoretical contextual reinterpretation.

## Where we left off

This closes out **Section 2: Sectioning & Structure** entirely — `<html>`/`<body>` + `lang`/`dir`, `<header>`/`<footer>`/`<main>`/`<nav>`/`<article>`/`<section>`/`<aside>` (including the article vs section vs div decision test applied to real Vue component habits), `<div>`'s correct use, heading hierarchy rules, `<hgroup>`, and the outline algorithm's historical/obsolete status.

Next up per the HTML reference doc: **Section 3 — Text Content** (`<p>`, `<blockquote>`, `<pre>`, `<hr>`, lists — `<ul>`/`<ol>`/`<li>`/`<dl>`/`<dt>`/`<dd>` — and inline text semantics like `<em>`, `<strong>`, `<small>`, `<s>`, `<cite>`, `<q>`, `<dfn>`, `<abbr>`, `<time>`, `<code>`, `<mark>`, `<bdi>`/`<bdo>`, `<span>`, and others).
