# Session Summary: `<head>` Contents (closes Section 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<title>`

```html
<title>Page Name — Site Name</title>
```
- Sets browser tab text, default bookmark name, and the clickable headline in search results
- Every page needs exactly one, unique per page — matters for SEO and usability with multiple tabs open

## `<base>`

```html
<base href="https://example.com/app/">
```
- Sets a default base URL that all relative URLs on the page resolve against
- Rare in practice — mainly for apps served/embedded from multiple different paths
- At most one per document, must appear before any element using a relative URL

## `<meta>` — important variants

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="description" content="...">
<meta http-equiv="refresh" content="30">
```
- **`viewport`** — essential for responsive design; without it, mobile browsers fake a desktop-width viewport and zoom out, breaking media queries
- **`name="description"`** — used by search engines for the snippet under the result link (SEO)
- **`http-equiv`** — mimics an HTTP header via markup (e.g. auto-refresh); mostly legacy, real server-side headers preferred

## `<link>` — rel types that matter

```html
<link rel="stylesheet" href="styles.css">
<link rel="icon" href="favicon.ico">
<link rel="canonical" href="https://example.com/page">
<link rel="preload" href="font.woff2" as="font" crossorigin>
<link rel="preconnect" href="https://fonts.googleapis.com">
```
- **`stylesheet`** — everyday CSS linking
- **`icon`** — favicon
- **`canonical`** — tells search engines the authoritative URL when content is reachable via multiple URLs (avoids duplicate-content SEO penalties)
- **`preload`** — fetches a resource early, before the parser would normally discover it (fonts, critical CSS/JS)
- **`preconnect`** — opens the network connection (DNS + TCP + TLS) to a domain early, saving latency on the first real request to it

## Script loading — `async` / `defer` / `type="module"` (the practically important part)

**No attribute (default):**
```html
<script src="app.js"></script>
```
Parsing stops completely during download + execution. Traditionally placed at the end of `<body>` to avoid blocking render.

**`defer`:**
```html
<script defer src="app.js"></script>
```
- Downloads without blocking parsing
- Executes after parsing finishes, before `DOMContentLoaded`
- Multiple deferred scripts execute in document order, guaranteed
- Safe default for most app scripts; can live in `<head>` instead of end of `<body>`

**`async`:**
```html
<script async src="analytics.js"></script>
```
- Downloads without blocking parsing
- Executes immediately whenever download finishes — could interrupt parsing, and executes in **no guaranteed order** relative to other scripts
- Good for fully independent scripts (analytics, ads); bad for anything needing DOM readiness or specific order relative to other scripts

**`type="module"`:**
```html
<script type="module" src="app.js"></script>
```
- Enables `import`/`export` syntax
- Deferred by default (non-blocking, executes after parsing — same behavior as `defer` without needing it explicitly)
- Runs in strict mode automatically
- Each module has isolated scope — no accidental global leakage between files

### Traced example: mixed defer/async ordering

```html
<script defer src="a.js"></script>
<script async src="b.js"></script>
<script defer src="c.js"></script>
```

**Guaranteed:** `a.js` executes before `c.js` — defer preserves relative document order between deferred scripts.

**NOT guaranteed:** `b.js` (async) could execute before both, between them, or after both — purely dependent on download speed relative to the others. No fixed position.

**Why this matters practically:** if an async script depends on something a deferred script sets up (a global variable, a DOM element), you get an intermittent bug — works in dev, randomly breaks in production when network timing shifts. This is a real, hard-to-reproduce bug class.

**Practical rule:** never use `async` for a script that depends on execution order relative to anything else — reserve it strictly for fully independent scripts.

## Where we left off

This closes out **Section 1: Document & Syntax Fundamentals** entirely — doctype/versioning, elements/tags/nodes (+ full parsing deep dive: tokenization, tree construction, error recovery), attributes, void elements/comments/whitespace/case sensitivity, character encoding & entities, and now `<head>` contents. Next up per the HTML reference doc: **Section 2 — Sectioning & Structure** (`<html>`, `<body>`, `<header>`, `<footer>`, `<main>`, `<nav>`, `<article>`, `<section>`, `<aside>`, `<div>` and when it's correct, heading elements, outline algorithm).
