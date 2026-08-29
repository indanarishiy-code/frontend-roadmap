# Session Summary: `<html>`/`<body>` and `lang`/`dir`

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<html>`

- The root element — everything else lives inside it
- Key attribute: `lang`
```html
<html lang="en">
```
- `lang` tells screen readers which pronunciation rules to use, tells browsers which spell-check dictionary to apply, and tells search engines/translation tools the content's language
- Relevant for multilingual target markets (Germany, NL, Malaysia, UAE) — `lang` often needs to be set per-page or per-element

## `<body>`

- Contains everything rendered/visible on the page
- Only one per document, comes after `<head>`
- Mostly a container — the real substance is what goes inside it (semantic sectioning elements vs `<div>`, covered next session)

## Setting `lang` programmatically for i18n

**Server-rendered / framework case (e.g. Next.js):**
```jsx
export default function RootLayout({ children, params: { locale } }) {
  return (
    <html lang={locale}>
      <body>{children}</body>
    </html>
  );
}
```
`lang` comes from the resolved locale via the i18n library (next-intl, next-i18next, Nuxt i18n, etc.) — set once per request, not manually toggled.

**Client-side only (SPA):**
```js
document.documentElement.lang = currentLocale; // e.g. "de", "ar"
```

**Per-element override** — for a single foreign phrase inside otherwise single-language content:
```html
<p>The German word <span lang="de">Schadenfreude</span> has no direct English equivalent.</p>
```

## `dir` attribute

```html
<html dir="ltr">   <!-- left-to-right: English, German, Dutch, most languages -->
<html dir="rtl">   <!-- right-to-left: Arabic, Hebrew, Farsi, Urdu -->
<html dir="auto">  <!-- browser detects direction from text content -->
```

`dir="rtl"` doesn't just reverse reading order — it flips the whole layout: default text alignment, scrollbar position, and inline content's visual start/end.

### Physical vs logical CSS properties — the real practical lesson

Physical properties (`margin-left`, `text-align: left`) do NOT auto-flip with `dir`. Logical properties do, automatically:

```css
/* Breaks in RTL — stays physically wrong */
.card { margin-left: 16px; text-align: left; }

/* Correct — flips automatically based on dir */
.card { margin-inline-start: 16px; text-align: start; }
```

**Rule for real work, especially UAE-market Arabic support:** use CSS logical properties (`margin-inline-*`, `padding-inline-*`, `border-inline-*`, `text-align: start/end`) throughout instead of `left`/`right`, paired with `dir` set correctly on `<html>`.

### `dir="auto"`

Useful for user-generated content of unknown language (comments, chat) — browser inspects the first strong directional character and sets direction per-element automatically.

### `<bdi>` / `<bdo>` — edge cases, not everyday tools
- `<bdi>` — isolates text so its directionality doesn't affect surrounding text (e.g. embedding an unknown-direction username inline in a sentence)
- `<bdo>` — forcibly overrides direction, ignoring natural text direction; rare

## `lang` + `dir` pairing rule for real i18n

In production, `lang` should only change when the actual content changes language — i.e., as part of a full locale switch (translated strings + `lang` + `dir` together). A direction-only toggle (e.g. a demo RTL preview button) shouldn't change `lang` if the content itself isn't translated.

## Exercise: profile card with RTL toggle

Built a profile card styled entirely with logical properties (`margin-inline`, `padding-inline`, `padding-block`, `border-inline-start`, `text-align: start`), plus a button toggling `document.documentElement.dir` between `ltr`/`rtl` live.

### Review — first pass
- **Correct:** logical properties used throughout, no physical `left`/`right` fallback — visually confirmed border/alignment flip on toggle
- **Missing:** `aria-pressed` on the toggle button (no state indication for assistive tech) — recurring accessibility pattern from the earlier toggle-list exercise
- **Missing:** `lang` never updated alongside `dir` on toggle — flagged as a real-world bug pattern (mismatched `lang`/`dir` signals to screen readers)

### Review — second pass (after fixes)
- `aria-pressed` correctly toggled via `button.setAttribute("aria-pressed", String(!isRTL))` — explicit `String()` wrapping is good practice
- `lang`/`dir` now toggle together (`en`/`ltr` and `ar`/`rtl` pairs)
- `type="button"` added — good defensive habit against accidental form submission
- **Nuance flagged (not a real bug):** the exercise's own code comment noted "only change lang if content itself is changing language" — correctly identifying the real-world rule, even though the demo's actual text content doesn't change language on toggle (acceptable for a direction-mechanics demo, not a misunderstanding)

## Where we left off

`<html>`/`<body>` + `lang`/`dir` slice closed, including a full exercise/review/revision cycle demonstrating solid grasp of CSS logical properties and the recurring `aria-pressed` accessibility pattern (applied correctly without prompting on the second pass). Next in Section 2: `<header>`, `<footer>`, `<main>`, `<nav>`, `<article>`, `<section>`, `<aside>`, `<div>` and when it's correct, heading elements, and the outline algorithm.
