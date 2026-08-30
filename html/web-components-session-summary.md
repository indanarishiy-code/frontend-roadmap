# Session Summary: Web Components (Section 9)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Custom Elements — defining new HTML tags

```js
class MyCounter extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<button>Count: 0</button>`;
    // ... event handling
  }
}
customElements.define('my-counter', MyCounter);
```
```html
<my-counter></my-counter>
```
Registers a genuinely new HTML tag backed by a JS class. Browser calls lifecycle callbacks automatically: `connectedCallback()` (added to DOM), `disconnectedCallback()` (removed), `attributeChangedCallback()` (watched attribute changes). Custom element names **must contain a hyphen** — a spec requirement so custom names never collide with future native HTML elements.

## Shadow DOM — real, native encapsulation

### The problem it solves
In regular DOM, all CSS is global by default. Frameworks invented userland workarounds (CSS Modules, Vue's `scoped`, styled-components) to simulate isolation. Shadow DOM does this natively at the browser level.

### Two boundaries it enforces
**1. Style encapsulation (both directions):**
```js
const shadow = this.attachShadow({ mode: 'open' });
shadow.innerHTML = `<style>p { color: red; }</style><p>I'm red</p>`;
```
Outside global CSS cannot reach in; inside shadow CSS cannot leak out. Both directions blocked.

**2. DOM query isolation:**
```js
document.querySelectorAll('p'); // does NOT find <p> inside a shadow root
```
A genuinely separate DOM tree, not just visually scoped.

### What still crosses the boundary
- **Inherited CSS properties** (`color`, `font-family`, `line-height`) — inheritance is a separate mechanism from selector matching
- **CSS custom properties (variables)** — explicitly designed to pierce shadow boundaries; the sanctioned way to theme a shadow-DOM component from outside:
```css
/* outside */ my-widget { --accent-color: purple; }
/* inside */  p { color: var(--accent-color); }
```

### Light DOM vs Shadow DOM vs Slots
- **Light DOM** — regular children written between a custom element's tags in actual HTML
- **Shadow DOM** — the internal, encapsulated tree the component renders for itself
- **Slots** — the mechanism projecting light DOM content into specific spots inside the shadow DOM tree. This is directly the same mental model as **Vue's `<slot>`** — Vue's slot system is a conceptual descendant of this native concept.

### `open` vs `closed` mode
```js
this.attachShadow({ mode: 'open' });   // element.shadowRoot works from outside
this.attachShadow({ mode: 'closed' }); // element.shadowRoot returns null
```
`open` used the vast majority of the time (DevTools, testing tools need access). `closed` is rare — some native browser elements (`<video>` controls) use closed shadow roots internally, which is why they can't be fully restyled with normal CSS.

### Event retargeting across the shadow boundary
```js
// inside shadow root
shadow.querySelector('button').addEventListener('click', (e) => {
  console.log(e.target); // <button> — correct, from inside
});
// outside, listening on the custom element
myWidget.addEventListener('click', (e) => {
  console.log(e.target); // <my-widget> — NOT the internal button!
});
```
Intentional: code outside a component shouldn't need to know its internal DOM structure. Breaks typical event-delegation patterns (`e.target.matches(...)`) if you're used to normal bubbling.

**Not all events cross the boundary** — standard UI events (`click`, `input`, `keydown`) are composed and do cross; custom events must opt in explicitly:
```js
this.dispatchEvent(new CustomEvent('my-event', {
  bubbles: true,
  composed: true, // required to cross the shadow boundary
}));
```
Forgetting `composed: true` is a common bug — event works when tested from inside, silently never reaches outside listeners.

## Declarative Shadow DOM — parse-time, not just alternate syntax

```html
<my-widget>
  <template shadowrootmode="open">
    <style>p { color: red; }</style>
    <p>Hi</p>
  </template>
</my-widget>
```
**The real distinction is timing, not syntax:** regular JS-based Shadow DOM doesn't exist until JS runs — on server-rendered pages, this causes a visible flash/delay before encapsulated content appears. Declarative Shadow DOM is created by the **HTML parser itself** at parse time, before any JS executes or hydrates — solving the SSR "flash of unencapsulated content" problem that JS-driven Shadow DOM couldn't avoid.

## Slots (recap)

```html
<my-widget>
  <h2 slot="title">Card Title</h2>
  <p>Default slot content</p>
</my-widget>
```
Named slots project specific content into specific placeholders; the unnamed default slot catches everything else — same model as Vue's slots.

## Does React use Shadow DOM? — No

React's model is built on the regular, single, global DOM tree via Virtual DOM diffing — no shadow root, no encapsulated subtree anywhere by default.

**Common source of confusion:** "Virtual DOM" (React's in-memory diffing mechanism) sounds similar to "Shadow DOM" (a real encapsulated DOM subtree) but they're unrelated concepts. React did briefly explore Shadow DOM support in past discussions/RFCs, and some React-based component libraries built as native Web Components do use real Shadow DOM under the hood even when consumed inside a React app — both are plausible reasons for encountering this before.

Vue can opt into Shadow DOM via `defineCustomElement()`, but Vue's default `scoped` styles are their own separate userland mechanism, not Shadow DOM.

## Frameworks that DO use Shadow DOM as their core model

**Lit** (Google-maintained, successor to Polymer):
```js
class MyCounter extends LitElement {
  static styles = css`p { color: red; }`;
  render() { return html`<p>Count: ${this.count}</p>`; }
}
```
Every Lit component is a native Custom Element with real Shadow DOM attached automatically — a thin, ergonomic layer over the raw Custom Elements/Shadow DOM APIs.

**Stencil** — a compiler (used by Ionic) that outputs real Web Components with Shadow DOM from a React-like JSX syntax, enabling the same component library to work identically across React, Vue, Angular, or plain HTML.

## Why React/Vue don't use Shadow DOM by default

1. **Cross-component theming conflict** — Shadow DOM blocks CSS cascade, but React/Vue apps rely on shared design systems (global CSS variables, Tailwind utilities) cascading freely across components. Would turn trivial global theming into a deliberate per-component opt-in tax.
2. **Performance model mismatch** — Virtual DOM diffing and Vue's reactivity are optimized around fast, direct DOM access; shadow boundaries add real overhead to cross-boundary queries/style computation at scale.
3. **Historical/ecosystem lock-in** — React and Vue predate stable, widely-supported Custom Elements/Shadow DOM. Their component models, DevTools, SSR strategies, and testing ecosystems were all built around plain DOM; retrofitting Shadow DOM as default now would be a massive breaking change for benefits already reasonably solved via CSS Modules/scoped styles.
4. **Historical SEO/SSR gaps** — search engines and SSR pipelines had inconsistent support for indexing/hydrating shadow-root content (part of why Declarative Shadow DOM had to be invented later).

**Honest summary:** not that Shadow DOM is worse — React/Vue's userland scoping gets "good enough" encapsulation without cross-boundary friction, while Lit/Stencil trade that convenience for true framework-agnostic reusability, which only matters for specific use cases (shared design systems across multiple frameworks).

## Practical takeaway

Very unlikely to hand-write Custom Elements/Shadow DOM in day-to-day Vue work. Real value here: recognizing Vue's slot system as a conceptual descendant of native Shadow DOM slots (good interview context), and knowing Web Components/Lit/Stencil exist specifically for framework-agnostic, cross-stack component libraries — relevant mainly if working at a company building shared design systems across multiple frontend frameworks.

## Where we left off

Section 9 (Web Components) covered in real depth: Custom Elements, Shadow DOM mechanics (style/DOM isolation, exceptions, light DOM/slots relationship, open/closed mode, event retargeting + `composed`), Declarative Shadow DOM's parse-time rationale, and a clear correction of the React/Shadow DOM misconception plus the frameworks that do use it (Lit, Stencil) and why React/Vue don't by default. No exercise built for this section.

Next up per the HTML reference doc: **Section 10 — Accessibility (ARIA + native semantics)** — ARIA roles, ARIA states/properties, implicit ARIA semantics, `tabindex`, focus management APIs, `alt`/`title` proper usage.
