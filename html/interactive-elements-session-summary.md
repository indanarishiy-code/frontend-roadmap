# Session Summary: Interactive Elements (Section 8)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<details>` / `<summary>`

Native, no-JS collapsible disclosure widget:
```html
<details>
  <summary>Click to expand</summary>
  <p>Hidden content revealed on click.</p>
</details>
```
`<summary>` is the always-visible clickable header; everything else is hidden until toggled. `open` boolean attribute starts it expanded:
```html
<details open>...</details>
```
Genuinely useful for FAQs, accordions — free keyboard accessibility and toggle behavior with zero JS.

## `<dialog>` — native modal/popup element

```html
<dialog id="myDialog">
  <p>Dialog content</p>
  <button onclick="document.getElementById('myDialog').close()">Close</button>
</dialog>
```
```js
dialog.showModal();  // true modal — traps focus, dims background, blocks outside interaction
dialog.show();        // non-modal — no focus trap, background still interactive
dialog.close();
```

**Key distinction — `showModal()` vs `show()`:** `showModal()` gives real modal behavior for free — focus trapping, `Escape` closes it, and a styleable `::backdrop`:
```css
dialog::backdrop {
  background: rgba(0, 0, 0, 0.5);
}
```
**Why this matters:** before `<dialog>`, building an accessible modal (focus trap, escape handling, correct ARIA) required significant custom JS or a library. Native `<dialog>` provides correct behavior out of the box — a genuinely interview-relevant, high-value native API.

## PopoverAPI — lighter-weight than `<dialog>`

```html
<button popovertarget="myPopover">Open popover</button>
<div id="myPopover" popover>
  Popover content
</div>
```
- `popovertarget` on the trigger links to the popover's `id`
- `popover` attribute marks the target as a popover (hidden until triggered)
- Good for tooltips, menus, non-modal dropdowns
- Automatic light-dismiss (click outside closes it) and correct layering above other content without manual `z-index` management
- `popovertargetaction` = `"show"` / `"hide"` / `"toggle"` (default) controls what the trigger does

## Invoker Commands — newer, less common

```html
<button command="show-popover" commandfor="myPopover">Open</button>
```
Declarative alternative to writing `addEventListener` JS for common interactions (opening dialogs/popovers, form submission) directly via HTML attributes. Still gaining browser support — worth knowing it exists, not yet reliable for broad compatibility.

## `<template>`

```html
<template id="rowTemplate">
  <tr><td class="name"></td><td class="value"></td></tr>
</template>
```
Content inside is **inert** — not rendered, not run (scripts don't execute, images don't load) — until explicitly cloned via JS:
```js
const template = document.querySelector('#rowTemplate');
const clone = template.content.cloneNode(true);
clone.querySelector('.name').textContent = 'Item';
document.querySelector('table').appendChild(clone);
```
Useful for reusable markup fragments stamped out repeatedly (rows, cards) without a full framework. Shows up less in actual Vue-based work since Vue handles templating itself, but worth recognizing.

## Practical priority

For modern frontend work, the genuinely high-value items here: **`<dialog>`** (real interview-relevant native modal, replaces a lot of custom modal JS) and **PopoverAPI** (newer but increasingly common for dropdowns/tooltips). `<details>`/`<summary>` is a nice free-accessibility win for simpler disclosure patterns. Invoker Commands and `<template>` are more "know it exists" level given current framework-heavy workflows.

## Where we left off

Section 8 (Interactive Elements) covered in full pass. No exercise built yet — could revisit later with a small `<dialog>`-based modal exercise if useful.

Next up per the HTML reference doc: **Section 9 — Web Components (native, framework-independent)** — Custom Elements, Shadow DOM, Declarative Shadow DOM, HTML Templates, Slots.
