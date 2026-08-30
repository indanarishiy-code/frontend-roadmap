# Session Summary: Metadata, SEO & Social (Section 11)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: viewport meta and `<link rel="canonical">` were already covered in the `<head>` contents session.*

## Open Graph tags

Control how a page appears when shared on social platforms (Facebook, LinkedIn, and others using the same protocol):
```html
<meta property="og:title" content="Frontend Roadmap Guide">
<meta property="og:description" content="A self-study guide for frontend interviews">
<meta property="og:image" content="https://example.com/thumbnail.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">
```
Without these, sharing a link falls back to whatever generic title/description the platform can scrape — often broken/unhelpful. `og:image` is the most commonly forgotten — no image means a bare text link, noticeably lower engagement.

## Twitter Card tags

Same idea, specifically for X/Twitter (historically didn't fully honor Open Graph):
```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Frontend Roadmap Guide">
<meta name="twitter:description" content="A self-study guide">
<meta name="twitter:image" content="https://example.com/thumbnail.jpg">
```
`summary_large_image` is the common card type (large preview image). Most sites just duplicate Open Graph values here rather than writing separate copy.

## JSON-LD structured data

Machine-readable metadata as JSON inside a `<script>` tag, telling search engines specific facts about page content:
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "How to Learn CSS Grid",
  "author": { "@type": "Person", "name": "Jane Doe" },
  "datePublished": "2026-01-15"
}
</script>
```
Powers rich search results — star ratings, recipe cook times, event dates, FAQ dropdowns directly in Google search. `schema.org` defines the `@type` vocabulary (Article, Product, Recipe, Event, FAQPage, etc.). Preferred over older inline microdata/RDFa since it's cleanly separated from visible markup.

## Robots meta directives

Controls search engine crawling/indexing at the page level:
```html
<meta name="robots" content="noindex, nofollow">
```
- **`noindex`** — exclude page from search results
- **`nofollow`** — don't follow links on this page for ranking purposes
- **`index, follow`** — default behavior, usually omitted entirely

**Practical relevance:** admin panels, internal dashboards, duplicate content variants, or CMS/dashboard work with public-facing preview URLs that shouldn't be indexed.

## Remaining resource hints (viewport/canonical/preload/preconnect covered earlier)

```html
<link rel="prefetch" href="/next-page.html">
<link rel="dns-prefetch" href="https://analytics-provider.com">
<link rel="modulepreload" href="app.js">
```
- **`prefetch`** — hints the browser to fetch a resource likely needed for a **future navigation** (not this page), low priority, during idle time. Different from `preload` (this page, urgent).
- **`dns-prefetch`** — lighter version of `preconnect`; only resolves DNS ahead of time, no full TCP/TLS handshake. Useful when unsure a connection will actually be needed.
- **`modulepreload`** — like `preload` but for ES modules specifically; also preloads/initializes the module's dependency graph, not just the file itself.

## Favicon/manifest links

```html
<link rel="icon" href="/favicon.ico">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/manifest.json">
```
`manifest.json` enables PWA features (add to home screen, app icon, splash screen) — to be covered in more depth when reaching the PWA section of the roadmap.

## Practical priority

- **Open Graph** — most likely to come up in real work (marketing/product will request this for link sharing)
- **Robots directives** — relevant for anything with staging/preview environments, matching past CMS/dashboard work
- JSON-LD and Twitter Cards — good to know exist, less frequently hand-written day-to-day (often handled by CMS/framework SEO plugins)

## Where we left off

Section 11 (Metadata, SEO & Social) covered in full — Open Graph, Twitter Cards, JSON-LD, robots directives, and remaining resource hints (`prefetch`, `dns-prefetch`, `modulepreload`), plus favicon/manifest links. No exercise built for this section.

Next up per the HTML reference doc: **Section 12 — Security-Relevant HTML** (`rel="noopener noreferrer"` — already touched on in Links & Navigation — `<iframe sandbox>` values — already touched on in Embedded Content — CSP interaction with inline `<script>`/`<style>`, `integrity`/`crossorigin` for Subresource Integrity, X-Frame-Options relevance to `<iframe>` embedding).
