# Session Summary: HTML Attributes

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What an attribute is

Extra information attached to an opening tag, `name="value"` form: `<img src="cat.jpg" alt="A cat">` — `src` and `alt` are attributes.

## Global attributes

Usable on any HTML element regardless of type: `id`, `class`, `style`, `title`, `lang`, `dir`, `tabindex`, `hidden`, `data-*`, `contenteditable`, `draggable`, `spellcheck`, etc.

## Boolean attributes

Presence = true, absence = false. No meaningful value to assign — you don't write `checked="false"` to turn something off.

```html
<input type="checkbox">          <!-- unchecked -->
<input type="checkbox" checked>  <!-- checked -->
```

Common beginner mistake: `checked="false"` still counts as checked, since the attribute is *present* regardless of string value. Other examples: `disabled`, `required`, `readonly`, `multiple`, `autofocus`.

**Toggling correctly in JS:** set the DOM property, not `setAttribute` with a string:
```js
button.disabled = true;   // correct
button.disabled = false;  // correct
button.setAttribute("disabled", "false"); // WRONG — still disabled
```

## `data-*` attributes

Spec-reserved namespace for custom data your JS/CSS needs, without inventing non-standard attributes:
```html
<div data-user-id="482" data-role="admin">
```
Read/write via `dataset` (kebab-case → camelCase):
```js
el.dataset.userId  // "482"
```

**Case sensitivity gotcha:** `el.dataset.user-id` is invalid JS (parsed as subtraction) — must use `el.dataset.userId`.

## Custom (non-`data-*`) attributes

Writing arbitrary undefined attribute names (e.g. `fooBar="1"`) is invalid HTML — doesn't error, but fails validation and isn't guaranteed stable across browsers. `data-*` exists as the spec-sanctioned escape hatch for this exact need. Exception: framework-specific attributes (`v-if`, `*ngFor`) are fine since compilers strip/transform them before the browser sees raw HTML.

## `data-*` vs ARIA — real distinction, not just similar prefixes

- `data-*`: pure JS-consumed data, zero effect on rendering/accessibility tree
- ARIA (`aria-expanded`, `aria-hidden`, etc.): directly affects the accessibility tree, screen readers read them
- Common mistake: using `data-*` for a state that assistive tech should also know about (e.g. a dropdown's open/closed state) — that should be `aria-expanded`, not a custom data attribute, when it affects what a screen reader announces

## `data-*` as CSS selector hook — legitimate pattern

```css
[data-status="active"] { color: green; }
```
Deliberate, widely-used pattern for state-based styling, not a hack.

## Attribute vs property — relevant for React/Vue debugging

`checked`, `value`, `disabled` etc. exist as both an HTML *attribute* (markup / `setAttribute()`) and a live DOM *property* (`element.checked`). They can desync — `input.checked = true` in JS updates the live property but not the HTML attribute. Explains "why does `outerHTML` show unchecked but the checkbox looks checked" type bugs.

## Attribute ordering

No spec-defined order; auto-formatters (Prettier, ESLint plugins) may enforce one — a tooling/style convention, not an HTML rule.

## Exercise: toggle-list widget

Built a `<ul>` of 3 items, each with a toggle button switching `data-status` between `active`/`inactive`, styled via CSS attribute selector.

### Review findings (strict senior-dev pass)

1. **Missed requirement:** `disabled` boolean attribute toggle was never implemented — without it, the toggle could flip infinitely instead of locking once active.
2. **Boolean attribute correctness (for the fix):** use `button.disabled = (li.dataset.status === "active")`, not `setAttribute("disabled", "false")`.
3. **Accessibility gaps:**
   - Color-only state indication (red/green) fails WCAG 1.4.1 (Use of Color) — colorblind users can't distinguish state. Needs a second signal (text, icon, border) alongside color.
   - Missing `aria-pressed` on the toggle button — screen reader users get no indication the button's state changed. Should toggle `aria-pressed="true"/"false"` alongside the visual state, with an initial `aria-pressed="false"` in markup.
4. **Document structure bug:** `<style>` and `<script>` were placed after `</html>`, which is invalid. This visually "worked" only because of the parser's error-recovery behavior (implicitly reinserting trailing content into the tree) — a direct callback to the parsing topics covered earlier. Correct placement: `<style>` in `<head>`, `<script>` at end of `<body>` (or in `<head>` with `defer`).

### Corrected pattern (conceptual, not to copy verbatim into notes)
```js
document.querySelectorAll("li button").forEach((button) => {
  button.addEventListener("click", () => {
    const li = button.parentElement;
    const isActive = li.dataset.status === "active";
    li.dataset.status = isActive ? "inactive" : "active";
    button.setAttribute("aria-pressed", !isActive);
    button.disabled = !isActive;
  });
});
```
Note: even this corrected version still relies partly on color for visual feedback — a fully accessible version would also change visible button text/icon to indicate state, not just `aria-pressed` for screen readers.

## Where we left off

Attributes slice closed, including a full exercise + strict review cycle. Remaining Section 1 slices not yet covered: void elements/comments/whitespace/case sensitivity, character encoding & entity references, `<head>` contents (`<title>`, `<base>`, `<meta>`, `<link>` rel types, script loading `async`/`defer`/`type=module`).
