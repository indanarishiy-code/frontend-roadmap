# Session Summary: Accessibility — ARIA, EAA Context, Tooling (Section 10)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What ARIA is

A set of HTML attributes telling assistive technology (mainly screen readers) about roles/states/properties that native HTML doesn't otherwise communicate — mainly for custom widgets with no native equivalent.

## First rule of ARIA: don't use it if you don't have to

Native HTML elements already have implicit ARIA semantics for free:
```html
<button>Click me</button>
```
Already has role `button`, is keyboard-focusable, responds to click AND keyboard (Enter/Space) — all automatic.

Compare:
```html
<div role="button" tabindex="0" onclick="...">Click me</div>
```
Requires manually adding role, manually making focusable, manually implementing keyboard activation — easy to get wrong (missed keyboard support, missed focus styling).

**Practical rule:** always prefer real `<button>`, `<a>`, `<input>` etc. over a styled `<div>` with ARIA bolted on, unless no native element genuinely fits.

## ARIA roles — categories

- **Landmark roles** — `navigation`, `main`, `banner`, `contentinfo` — mostly map to elements already covered (`<nav>` implicitly has role `navigation`, `<main>` implicitly has role `main`); rarely need to add explicitly
- **Widget roles** — `button`, `checkbox`, `tab`, `slider`, `dialog` — for custom interactive components with no native equivalent
- **Document structure roles** — `heading`, `list`, `listitem` — mostly redundant if already using `<h1>`–`<h6>`, `<ul>`, `<li>`
- **Live region roles** — `alert`, `status`, `log` — for dynamically updating content that should be announced without the user navigating to it

## ARIA states and properties

```html
<button aria-pressed="false">Toggle</button>
<button aria-expanded="false" aria-controls="menu">Menu</button>
<div aria-hidden="true">Decorative, skip me</div>
<input aria-describedby="hint">
<p id="hint">Must be at least 8 characters</p>
```
- **`aria-expanded`** — announces open/closed state for collapsible things (accordions, dropdowns)
- **`aria-hidden="true"`** — hides purely decorative content from the accessibility tree (icons, decorative images) — never use on something visually visible and meaningful (creates a mismatch between what's seen and announced)
- **`aria-live="polite"`** — marks a region where dynamic updates get announced without moving focus (e.g. "3 items added to cart" toast)
- **`aria-describedby`/`aria-labelledby`** — link an element to other elements that describe/label it, by `id` reference

## `tabindex`

```html
<div tabindex="0">Now focusable, in natural tab order</div>
<div tabindex="-1">Focusable only via JS (element.focus()), not via Tab key</div>
<div tabindex="3">Forces a specific tab order — AVOID</div>
```
- **`0`** — makes a normally non-focusable element focusable in natural document order — legitimate for genuine custom interactive elements
- **`-1`** — removed from tab order but still programmatically focusable — used heavily for focus management (e.g. focusing a modal/error message via JS after it appears)
- **Positive values (1, 2, 3...)** — override natural tab order — actively avoid, widely considered an anti-pattern, creates a confusing/hard-to-maintain tab sequence

## Focus management APIs

```js
element.focus();
element.blur();
document.activeElement;
```
Important for custom modals: focus should move in when a modal opens (first focusable element or close button) and return to the trigger when it closes — otherwise keyboard/screen reader users lose their place. Native `<dialog>`'s `showModal()` (Section 8) handles this automatically.

## `alt` vs `title`

- **`alt`** — required on every `<img>`; describes content/purpose for screen readers and load failures; `alt=""` for purely decorative images
- **`title`** — hover tooltip; **not reliably announced by screen readers, inaccessible on touch devices** (no hover). Never a substitute for proper labeling or `alt` text — supplementary hint at best.

## Real-world context: why this matters for your target markets (verified via search)

**European Accessibility Act (EAA)** — in enforcement since June 28, 2025, across all 27 EU member states (including Germany and Netherlands), requiring compliance with WCAG 2.1 Level AA and EN 301 549.

- **Enforcement is active, not theoretical:** Germany's Bundesnetzagentur investigating complaints; Netherlands' ACM prioritizing e-commerce/banking platforms for early enforcement; first EAA lawsuits filed in French Commercial Court November 2025
- **Penalties are real:** up to €100,000 or 4% of annual revenue for non-compliance
- **Legacy product nuance:** any new features/updates to existing features must meet accessibility requirements even in legacy products as of June 28, 2025; if updates significantly affect functionality, a full accessibility review may be required. Fully untouched legacy apps are exempt only until June 28, 2030.
- **Practical implication:** even "internal-feeling" tools become in-scope the moment they're actively developed further — directly relevant to internal SPA/CMS work like what's been done historically, once EU market exposure or public-sector contracts are involved

**Conclusion:** accessibility skill in job postings for DE/NL markets reflects genuine legal exposure companies are hiring to mitigate, not just best-practice signaling.

## Framework support and tooling landscape (verified via search)

**Frameworks themselves don't enforce accessibility automatically** — React, Vue, Angular all output raw HTML, so the same ARIA/semantic rules apply regardless of framework. The real value is in tooling layered on top.

### Linting (catches issues while coding, in-editor)
- **React:** `eslint-plugin-jsx-a11y` — static JSX analysis, flags missing `alt`, missing label association, invalid ARIA values, `<div onClick>` without keyboard handler, etc. Included by default in many starter templates (e.g. Create React App) in recommended mode.
- **Vue:** `eslint-plugin-vuejs-accessibility` — direct Vue equivalent, checks accessibility rules within `.vue` files.
- **Cross-framework:** **axe Accessibility Linter** (VS Code extension) — covers HTML, Angular, React, Markdown, Vue, React Native, Liquid templates; zero-config.

### Runtime testing (catches issues in actual rendered DOM — linting alone can't catch everything)
- **`axe-core`** (Deque Labs) — the underlying engine most tools are built on
- **`@axe-core/react`** — audits rendered React output, logs issues to Chrome DevTools console
- Similar axe-core-based wrapper integrations exist for other frameworks

### Browser-based auditing (framework-independent)
- **Lighthouse** (built into Chrome DevTools) — generates an accessibility score/report, no install needed
- **axe DevTools** (Chrome extension) — more detailed developer-focused auditing

### CI/CD integration
- Tools like Chromatic can block PR merges on new/reintroduced accessibility violations — accessibility as an enforced quality gate, similar to test/lint requirements in modern pipelines

## Practical recommendation

Given Vue is the primary stack: install **`eslint-plugin-vuejs-accessibility`** in the sandbox repo — genuinely high-leverage, flags real issues automatically going forward, turning "I know the rules" into "my tooling enforces the rules" — a legitimate, honest talking point for interviews.

## Where we left off

Section 10 (Accessibility) covered in depth: ARIA fundamentals, roles/states/properties, tabindex, focus management, alt/title — plus real-world grounding via EAA legal context (verified current as of search) and the concrete tooling landscape across React/Vue/Angular. Honest acknowledgment that hands-on accessibility experience is limited from past internal-dashboard/CMS work, with EAA context now providing clear motivation to close that gap. Suggested next step (not yet done): install `eslint-plugin-vuejs-accessibility` in sandbox and/or retrofit an existing sandbox component with proper ARIA as a hands-on exercise.

Next up per the HTML reference doc: **Section 11 — Metadata, SEO & Social** (`<meta name="description">`, Open Graph tags, Twitter Card tags, `<link rel="canonical">`, favicon/manifest links, JSON-LD structured data, robots meta directives).
