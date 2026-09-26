# Session Summary: DOM Traversal & Events (JS Section 14, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. This is the first of multiple planned sessions on Section 14 given its size.*

## DOM traversal and manipulation

```js
document.querySelector('.card');          // first match
document.querySelectorAll('.card');        // NodeList — STATIC snapshot, not live
document.getElementById('header');          // single element by id, fastest lookup
document.getElementsByClassName('card');    // HTMLCollection — LIVE, updates automatically

const el = document.createElement('div');
el.textContent = "Hello";
parent.appendChild(el);
el.remove(); // modern, simpler alternative to parent.removeChild(el)
```
**Genuinely important distinction:** `querySelectorAll()` returns a static snapshot that doesn't update if the DOM changes afterward. `getElementsByClassName()`/`getElementsByTagName()` return a live collection that automatically reflects DOM changes. Real gotcha: looping over a live collection while adding/removing matching elements inside that loop can cause elements to be skipped or behave unexpectedly, since the collection changes size mid-iteration.

## Event object — `target` vs `currentTarget`, prevention

```js
button.addEventListener('click', (event) => {
  event.preventDefault();   // stops default browser behavior (form submit, link navigation)
  event.stopPropagation();   // stops the event continuing to bubble/capture further
  event.target;                // the actual element that triggered the event
  event.currentTarget;          // the element the LISTENER is attached to
});
```
**`target` vs `currentTarget`:** if a listener is on a parent `<div>` but a `<button>` inside it is clicked, `target` is the button, `currentTarget` is the div. Only identical when clicking directly on the listener's own element — the divergence once bubbling is involved is exactly what makes event delegation possible.

## Event bubbling and capturing — the two-phase mechanism

```html
<div id="outer"><div id="inner"><button id="btn">Click</button></div></div>
```
```js
outer.addEventListener('click', () => console.log('outer'));
inner.addEventListener('click', () => console.log('inner'));
btn.addEventListener('click', () => console.log('button'));
```
Clicking the button logs `button`, `inner`, `outer` — the **bubbling phase** (default): fires on the target first, then travels up through ancestors.

**Capturing phase — real but less commonly used:** every event actually travels through the DOM tree twice — first *down* from root to target (capture), then back *up* (bubble). `{ capture: true }` makes a listener fire during the capture phase, running *before* the target's own listener.
```
Full order with both: outer(capture) → inner(capture) → button(bubble) → inner(bubble) → outer(bubble)
```
**Practical relevance:** rare in typical app code, but some libraries use capture-phase listeners specifically to intercept an event before a component's own handler can call `stopPropagation()` on it — otherwise impossible to work around from a bubble-phase listener.

## Event delegation — the practical payoff of understanding bubbling

```js
// One listener on the parent, using event.target to identify what was clicked
document.querySelector('.table-body').addEventListener('click', (event) => {
  const row = event.target.closest('.row');
  if (row) handleRowClick(row);
});
```
**Why this matters for dashboard work specifically:** for a data table with rows added/removed dynamically (filtering, pagination, live updates), individual per-row listeners mean new rows silently have no listener unless re-attached on every update. A single delegated listener on the parent, relying on bubbling + `event.target`, works automatically even for rows that don't exist yet at attach time. `closest()` is the key method — walks up from the clicked element to the nearest ancestor matching a selector, needed since `event.target` might be a `<span>`/`<td>` inside the row, not the row itself.

## Custom events

```js
const event = new CustomEvent('item-selected', {
  detail: { id: 42, name: "Widget" },
  bubbles: true // ties back to the Shadow DOM composed/bubbles discussion
});
element.dispatchEvent(event);
element.addEventListener('item-selected', (e) => e.detail.id); // 42
```
Lets you dispatch application-specific events with custom payload data via `detail` — useful for decoupled component communication in vanilla JS/Web Components. In Vue specifically, Vue's own emit system is typically used instead for component communication.

## Does Vue's `@click` stop event bubbling by default? — No

```vue
<div @click="handleOuter">
  <button @click="handleInner">Click</button>
</div>
```
Clicking the button fires `handleInner`, THEN bubbles up and also fires `handleOuter` — identical bubbling behavior to vanilla JS. Vue's template event syntax is a convenient wrapper around `addEventListener`, not a behavior change to propagation.

**Vue's explicit modifiers — opt-in only:**
```vue
<button @click.stop="handleClick">Click</button>       <!-- calls stopPropagation() automatically -->
<button @click.prevent="handleSubmit">Submit</button>   <!-- calls preventDefault() automatically -->
```
`.stop`/`.prevent` are syntactic sugar for calling the native methods yourself — purely template convenience, not a Vue-specific behavior change. Without `.stop`, bubbling proceeds completely normally.

**Real, common dashboard scenario this affects — clickable row containing an action button:**
```vue
<tr @click="selectRow(item)">
  <td>{{ item.name }}</td>
  <td><button @click="deleteItem(item)">Delete</button></td>
</tr>
```
Clicking "Delete" fires `deleteItem`, then bubbles up and ALSO fires `selectRow` — likely both deleting the item and triggering row-select, not the intended behavior. **Fix:**
```vue
<button @click.stop="deleteItem(item)">Delete</button>
```

**Other modifiers, same "explicit opt-in" pattern:**
```vue
<div @click.self="handleClick">   <!-- only fires if click target IS this element, not a bubbled child click -->
<input @keyup.enter="submit">       <!-- only fires for Enter key -->
<div @click.once="handleOnce">     <!-- listener auto-removes after firing once -->
```
`.self` on the parent row is an alternative one-line fix to the same row/button problem, instead of adding `.stop` to every child button individually — both solve the same problem from different ends.

## Where we left off

First slice of Section 14 (The DOM & Browser APIs) covered: DOM traversal/manipulation (live vs static collections), the event object (`target`/`currentTarget`), bubbling/capturing as the real two-phase mechanism, event delegation with `closest()` (directly relevant to dynamic dashboard tables), custom events, and a thorough, practically-grounded discussion of Vue's bubbling behavior and modifiers (`.stop`, `.prevent`, `.self`) with a real row/button scenario matching actual dashboard work.

**Remaining Section 14 slices (planned for follow-up sessions):** `fetch()` API, Headers/Request/Response objects, localStorage/sessionStorage/cookies, history API for SPA routing, IntersectionObserver/ResizeObserver/MutationObserver, requestAnimationFrame, Web Workers, Intl API.
