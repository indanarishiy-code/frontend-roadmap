# Session Summary: Security-Relevant HTML (Section 12)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: `rel="noopener noreferrer"` was covered in Links & Navigation, `<iframe sandbox>` values were covered in Embedded Content.*

## Content Security Policy (CSP) — the concept

Usually set via an HTTP response header (or `<meta http-equiv="Content-Security-Policy">`), tells the browser which sources of content are allowed to load/execute on the page:
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' https://trusted-cdn.com">
```
**Why it exists:** primarily defends against **XSS (Cross-Site Scripting)** — if an attacker injects malicious `<script>` content into a page (via a comment field, URL parameter, etc.), a strict CSP can prevent that injected script from executing even if it made it into the DOM.

## The HTML consequence: inline scripts/styles blocked by default under strict CSP

```html
<script>alert('hi')</script>       <!-- inline script -->
<div style="color: red">...</div>  <!-- inline style -->
```
Under a strict CSP, inline scripts and inline styles are blocked by default — because inline content is exactly what an XSS injection looks like. **Real practical consequence:** on a strictly-CSP'd project, you can't write inline `<script>` blocks or `onclick="..."` handlers — everything must be in external `.js` files, or explicitly allowed via:
- **`nonce`** — a random per-request token
- **`hash`** — a hash of the exact script content

```html
<script nonce="a1b2c3">console.log('allowed');</script>
```
```
Content-Security-Policy: script-src 'nonce-a1b2c3'
```
Only a script tag with the matching nonce executes — how modern strict-CSP sites (banks, enterprise apps) still safely allow specific inline scripts.

## `integrity` / `crossorigin` — Subresource Integrity (SRI)

Verifies a third-party file (CDN script/stylesheet) wasn't tampered with (compromised CDN, man-in-the-middle):
```html
<script src="https://cdn.example.com/library.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
        crossorigin="anonymous"></script>
```
- **`integrity`** — cryptographic hash of expected file content. Browser downloads, hashes it itself, compares. Mismatch → **browser refuses to execute the file entirely**.
- **`crossorigin="anonymous"`** — required alongside `integrity` for cross-origin resources; allows the browser to actually read the response to compute the hash (cross-origin responses are otherwise opaque by default).

**Practical relevance:** loading any library from a public CDN (jsDelivr, cdnjs, unpkg) rather than bundling it — adding `integrity`+`crossorigin` is a genuine real-world security practice; most CDN docs generate the correct hash to copy-paste.

## X-Frame-Options and `<iframe>` embedding

An HTTP header the **embedded** page sends (not something written in the embedding page's HTML), but it directly determines whether an `<iframe>` will render at all:
```
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
```
- `DENY` — no page anywhere can embed it in an `<iframe>`; browser refuses to render, shows blank frame
- `SAMEORIGIN` — embedding only allowed from pages on the same origin

**Why it exists:** prevents **clickjacking** — tricking a user into clicking something inside an invisible/disguised iframe that's actually a different site's sensitive action (e.g. an invisible iframe of a bank's "transfer funds" button positioned over a fake game button).

**Practical debugging relevance:** if embedding another site via `<iframe>` mysteriously renders blank, check that site's `X-Frame-Options` (or the newer, more flexible `Content-Security-Policy: frame-ancestors` directive) response header first — it's not a bug in your code, the target site is actively blocking embedding.

## Practical takeaway

Two genuinely real "why isn't this working" debugging scenarios worth remembering: **CSP breaking inline scripts/styles**, and **X-Frame-Options blocking iframe embeds**. Both look like mysterious bugs until you know to check the relevant header/policy.

## Where we left off

Section 12 (Security-Relevant HTML) covered in full — CSP and its inline script/style blocking behavior (with nonce/hash as the sanctioned workaround), Subresource Integrity (`integrity`/`crossorigin`), and X-Frame-Options' role in iframe embedding and clickjacking prevention.

Next up per the HTML reference doc: **Section 13 — Internationalization** (`lang` attribute — already covered in depth — `dir` attribute — already covered in depth — character encoding edge cases, `<bdi>`/`<bdo>` — already briefly covered).
