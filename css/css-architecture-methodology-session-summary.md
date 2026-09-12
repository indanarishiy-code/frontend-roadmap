# Session Summary: CSS Architecture & Methodology (Section 15)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## BEM (Block Element Modifier)

```css
.card { }              /* Block */
.card__title { }        /* Element — __ separator */
.card__title--large { } /* Modifier — -- separator */
.card--featured { }     /* Modifier on the block itself */
```
**Problem solved:** flat class names in large codebases inevitably collide/become ambiguous. BEM makes class names self-documenting about structural relationship — you can tell `.card__title` belongs to `.card` from the name alone, no need to check HTML nesting. **Still genuinely relevant** in a component-framework world — many teams use BEM naming within a single component's scoped styles purely for clarity, even without true global collision risk.

## Utility-first vs component-scoped vs vanilla CSS — real architectural tradeoff

```html
<!-- Utility-first -->
<div class="flex items-center gap-4 p-4 rounded-lg bg-white shadow">
<!-- Component-scoped (Vue SFC) -->
<div class="card">
<style scoped>.card { display: flex; align-items: center; gap: 1rem; padding: 1rem; }</style>
```
- **Utility-first** — styling in markup, near-zero custom CSS, fast to build/iterate; markup gets visually dense, some argue less semantically readable
- **Component-scoped** — cleaner separation of concerns; requires naming/maintaining classes, can duplicate patterns without discipline
- **Vanilla/global CSS** — most flexible, doesn't scale to large teams without a naming convention (BEM) or scoping mechanism (global collision problem, Section 3)

**Honest take:** not a settled debate — genuinely still contested industry-wide, depends on team size/design system maturity/existing conventions. Vue background likely means most familiarity with component-scoped; Tailwind shows up extremely often across all four target markets' job postings — worth hands-on comfort with both.

## CSS Modules

```css
/* Button.module.css */
.button { background: blue; }
```
```jsx
import styles from './Button.module.css';
<button className={styles.button}>Click</button>
```
Build tool (webpack, Vite) rewrites `.button` into a globally-unique hashed class name at build time — the actual mechanism behind "scoped styles," distinct from Vue's `scoped` attribute (data-attribute + selector rewriting, not literal Shadow DOM — covered in the Web Components session). Common in React specifically, since React has no built-in scoping mechanism like Vue SFCs do.

## CSS-in-JS — historical context and current status shift

```jsx
const Button = styled.button`background: blue; padding: 8px 16px;`;
```
**Historical peak:** styled-components/Emotion very popular in React ~2018-2022 (dynamic prop-based styles, automatic scoping, colocation with component logic).

**Current status (important, since this has shifted):** fallen out of favor relative to peak popularity, primarily due to **runtime performance costs** — generating/injecting styles via JS at runtime is measurably slower than static CSS, a bigger concern as Core Web Vitals became central to frontend priorities. Many teams migrated to CSS Modules, Tailwind, or newer zero-runtime alternatives (Vanilla Extract, Panda CSS) that keep the DX benefits (colocation, type safety) without runtime cost. Worth knowing for interviews — assuming CSS-in-JS is still the default modern choice is somewhat dated.

## Design tokens and custom properties

```css
:root {
  --color-primary: #3b82f6;
  --spacing-sm: 8px;
  --spacing-md: 16px;
}
```
A "design token" is a named, centralized value representing a design decision — the token is the abstraction; CSS custom properties are the web implementation mechanism. Tokens are often defined once in a design tool (Figma) or JSON/YAML source of truth, compiled into CSS custom properties (or platform-specific formats for iOS/Android) — how design and engineering stay synchronized across platforms, and the mechanism enabling consistent theming (ties to `color-mix()` from Section 1).

## ITCSS — brief overview

**Inverted Triangle CSS** — organizes stylesheets generic-to-specific: Settings → Tools → Generic → Elements → Objects → Components → Utilities. More relevant to large vanilla-CSS-at-scale or Sass-based design systems than typical component-framework projects, where component boundaries already provide natural organization. Know the name/concept for recognition, not likely to implement from scratch in Vue work.

## Other architectures beyond BEM

**OOCSS (Object-Oriented CSS)** — BEM's conceptual predecessor:
```css
.btn { padding: 8px 16px; border-radius: 4px; }   /* structure */
.btn-primary { background: blue; color: white; }   /* skin */
```
Separates structural styles from visual/skin styles so they mix-and-match independently (classic "media object" pattern). BEM = OOCSS's philosophy + a stricter naming convention.

**SMACSS (Scalable and Modular Architecture for CSS)** — five categories: Base, Layout, Module, State, Theme.
**Genuinely useful idea that outlived the full methodology:** the **State category** — separating "what this looks like" classes from "what state this is currently in" classes (`.is-active`, `.is-loading`) is still widely used even by teams not following SMACSS as a whole. Connects directly to the `data-*` state-vs-style distinction from the Selectors session — SMACSS's `.is-*` convention and `data-state="..."` solve the same underlying problem with different mechanisms.

**Atomic CSS vs "utility-first" — a real distinction worth knowing:** Atomic CSS (academic/original concept, "functional CSS") predates Tailwind — extremely granular single-property classes (`.mt-10`, `.p-1`) covering a comprehensive combinatorial set upfront. **Tailwind is a specific, opinionated implementation** adding a constrained design-token scale and a build step generating only used classes. People often say "Tailwind" meaning "utility-first/atomic CSS" generally — the terms have real lineage, not Tailwind's own invention.

**CUBE CSS (Composition, Utility, Block, Exception)** — genuinely current, gained traction in recent years. Deliberately *combines* approaches: utility classes for common small patterns (spacing, flex alignment), BEM-like Block classes for reusable components, explicit Exception classes/attributes for one-off deviations. **Why it matters:** reflects current pragmatic industry consensus that pure utility-first OR pure component-scoped-only is often too rigid — most real codebases blend strategies anyway, and CUBE CSS formalizes that blend rather than treating it as accidental.

## Practical interview takeaway

BEM and utility-first (Tailwind) are the two most likely to come up directly given current job posting frequency across target markets. OOCSS/SMACSS worth recognizing by name (common "have you heard of" breadth checks). CUBE CSS worth mentioning to demonstrate awareness of current evolving thinking rather than treating this as a solved 2015-era debate.

## Where we left off

Section 15 (CSS Architecture & Methodology) covered in full, including a dedicated follow-up on architectures beyond BEM (OOCSS, SMACSS, Atomic CSS vs Tailwind distinction, CUBE CSS). The CSS-in-JS status shift and utility-first/component-scoped tradeoff flagged as the most currently relevant, interview-worthy pieces.

Next up per the CSS reference doc: **Section 16 — Performance-Relevant CSS** (critical rendering path — already covered in depth in HTML Section 14 — critical CSS extraction/inlining, selector performance, `contain` property, `content-visibility`).
