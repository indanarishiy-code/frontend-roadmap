# Session Summary: Forms — Structure & Input Types (Section 7, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. This is the first of multiple planned sessions on Forms, given the size of this section.*

## `<form>` — core attributes

```html
<form action="/submit" method="post" enctype="multipart/form-data" novalidate autocomplete="on">
```
- **`action`** — URL the form submits to
- **`method`** — `get` (data in URL query string, non-sensitive read requests) or `post` (data in request body, for state changes/sensitive data)
- **`enctype`** — encoding on submit; `multipart/form-data` required for file uploads, otherwise default (`application/x-www-form-urlencoded`) is fine
- **`novalidate`** — disables browser's built-in validation UI (for fully custom JS validation)
- **`autocomplete`** — `on`/`off` at form level; individual fields typically need their own specific tokens (covered in a later slice)

## `<label>` — non-negotiable for accessibility

```html
<label for="email">Email</label>
<input type="email" id="email">
```
`for`/`id` pairing makes the label clickable (focuses input) and lets screen readers announce the field's purpose. Alternative — wrap input inside label, skips needing matching ids:
```html
<label>
  Email
  <input type="email">
</label>
```
**Never use `placeholder` as a substitute for `<label>`** — placeholder text disappears once typing starts and isn't reliably announced as a label by all screen readers.

## `<fieldset>` / `<legend>`

Groups related fields with a group-level label:
```html
<fieldset>
  <legend>Shipping Address</legend>
  <label for="street">Street</label>
  <input type="text" id="street">
</fieldset>
```
Especially important for radio button groups — `<legend>` announces "what question is this group answering," which individual `<label>`s per radio can't convey alone.

## Basic input types

```html
<input type="text">
<input type="email">
<input type="password">
<input type="number">
<input type="checkbox">
<input type="radio" name="plan" value="basic">
<input type="radio" name="plan" value="pro">
```
Radios in the same group need the **same `name`** to be mutually exclusive — a common beginner bug is giving each radio a different `name`, silently breaking "only one selected" behavior.

## More input types

```html
<input type="tel">
<input type="url">
<input type="search">
<input type="date">
<input type="time">
<input type="datetime-local">
<input type="month">
<input type="week">
<input type="color">
<input type="range" min="0" max="100">
<input type="file" accept=".jpg,.png" multiple>
<input type="hidden" name="csrf_token" value="...">
```
- **`tel`/`url`/`email`/`search`** — trigger different mobile on-screen keyboards (numeric keypad for `tel`, visible `@` for `email`) — real, practical mobile UX win just from picking the right type
- **`date`/`time`/`datetime-local`** — native pickers, no JS date library needed for basic cases
- **`file`** — `accept` restricts file types shown (still needs server-side validation, it's just a UI hint); `multiple` allows multi-select
- **`hidden`** — submitted but not shown/editable (tokens, IDs)
- **`range`** — slider; `min`/`max`/`step` control behavior

## `<textarea>`, `<select>`, `<datalist>`

```html
<textarea rows="4" cols="50" maxlength="500"></textarea>

<select>
  <optgroup label="Fruits">
    <option value="apple">Apple</option>
    <option value="banana">Banana</option>
  </optgroup>
</select>

<input list="browsers" name="browser">
<datalist id="browsers">
  <option value="Chrome">
  <option value="Firefox">
</datalist>
```
**`<datalist>`** — turns a plain text `<input>` into an autocomplete-suggestion field without a JS-built dropdown; unlike `<select>`, user can still type anything, not restricted to the given options.

## `<button>` — the type attribute gotcha

```html
<button type="submit">Submit</button>
<button type="button">Just a click handler</button>
<button type="reset">Clear form</button>
```
**Real gotcha:** a `<button>` inside a `<form>` with no `type` specified defaults to `type="submit"`. Common bug: adding a button for an unrelated JS action inside a form, forgetting `type="button"`, and it accidentally submits the form on click.

## `<output>`, `<progress>`, `<meter>`

```html
<output name="result" for="a b">42</output>
<progress value="70" max="100"></progress>
<meter value="6" min="0" max="10">6 out of 10</meter>
```
- **`<output>`** — displays a calculation result (e.g. live-updating sum from two inputs)
- **`<progress>`** — indeterminate/determinate progress (loading, upload %) — implies "this will eventually complete"
- **`<meter>`** — a scalar measurement within a known range (disk usage, rating, score) — different from `<progress>`, no completion implication

## Where we left off

This covers form structure basics and the full input type surface. No exercise built yet for this part — could revisit with a small accessible form exercise later.

**Remaining Section 7 slices (planned for follow-up sessions):**
- Validation attributes (`required`, `pattern`, `min`/`max`, `step`, `minlength`/`maxlength`)
- **Constraint Validation API, `ValidityState`** — the original topic that prompted this whole study series
- FormData API
- Autocomplete/`autocomplete` token values in detail
