# Session Summary: Links & Navigation (Section 4)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<a>` — core attributes

```html
<a href="https://example.com">Visit site</a>
```

**`href`** — destination: absolute URL, relative path, fragment (`#section`), or special scheme (`mailto:`, `tel:`). No `href` at all means the element isn't actually a link — not keyboard-focusable, not exposed as a link to screen readers, just styled text.

**`target`**
```html
<a href="..." target="_blank">Opens in new tab</a>
```
`_blank` (new tab), `_self` (default, same tab), `_parent`/`_top` (mainly relevant in iframe contexts).

**`rel`** — relationship to the linked resource:
```html
<a href="https://external-site.com" target="_blank" rel="noopener noreferrer">External link</a>
```
- **`noopener`** — critical security practice. Without it, a `target="_blank"` link gives the opened page a `window.opener` reference back to your original tab, which its JS could exploit to redirect your tab (a known phishing technique). Always pair `target="_blank"` with `rel="noopener"`.
- **`noreferrer`** — also blocks the `Referer` header from being sent to the destination (privacy)
- **`nofollow`** — tells search engines not to pass ranking credit through the link (sponsored/untrusted content)
- **`sponsored`** / **`ugc`** — newer SEO-specific values for paid links / user-generated content links

**`download`**
```html
<a href="report.pdf" download>Download Report</a>
<a href="report.pdf" download="my-report.pdf">Download Report</a>
```
Forces download instead of navigation; optional value renames the downloaded file.

**`ping`** — fires a background POST request to given URL(s) on click, for click-tracking without JS. Rare in practice — most analytics use JS event listeners instead.

## URL schemes

```html
<a href="mailto:someone@example.com">Email us</a>
<a href="mailto:x@y.com?subject=Hello&body=Hi">Email with prefill</a>
<a href="tel:+491234567890">Call us</a>
```
- `mailto:` opens the default email client, pre-filled; supports `subject`/`body` query params
- `tel:` triggers a phone call on mobile devices

**`javascript:` — should never be used:**
```html
<a href="javascript:doSomething()">Click me</a>   <!-- avoid -->
```
Reasons to avoid: breaks progressive enhancement (nothing happens if JS fails), breaks right-click "open in new tab," and is a known XSS injection vector if the URL is ever built dynamically from user input. If there's no real navigation, use a `<button>` with a click handler instead — `<a>` implies "this goes somewhere."

## Fragment identifiers

```html
<a href="#pricing">Jump to pricing</a>
...
<section id="pricing">...</section>
```
Navigates to the element with matching `id` on the same page, no JS required.

**Anti-pattern:** `href="#"` with no real id — looks like a link, does nothing meaningful, often misused as a placeholder for "this should trigger JS." Same underlying problem as `javascript:` — should be a `<button>` instead.

## Key practical takeaways

1. **Security:** always pair `target="_blank"` with `rel="noopener noreferrer"`.
2. **Semantics/accessibility:** use `<button>`, not `<a href="#">` or `javascript:`, whenever nothing is actually navigating — `<a>` should always represent real navigation.

## Where we left off

Section 4 (Links & Navigation) covered in full: `<a>` attributes (`href`, `target`, `rel` values, `download`, `ping`), URL schemes (`mailto:`, `tel:`, `javascript:` and why to avoid it), and fragment identifiers.

Next up per the HTML reference doc: **Section 5 — Embedded Content** (`<img>` with `srcset`/`sizes`/`loading`/`decoding`/`fetchpriority`, `<picture>`/`<source>`, `<audio>`/`<video>`/`<track>`, `<iframe>` with `sandbox`/`allow`/`referrerpolicy`, `<embed>`/`<object>`/`<param>`, `<map>`/`<area>`, `<canvas>`, SVG embedding).
