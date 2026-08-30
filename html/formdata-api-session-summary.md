# Session Summary: FormData API (Section 7, Part 3)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What FormData is

A built-in JS object that captures form field values as key-value pairs automatically, without manually reading each input:

```js
const form = document.querySelector('form');
const formData = new FormData(form);
```

Pass a `<form>` element to the constructor — it collects every named field's current value using each input's `name` attribute as the key.

**Important distinction:** `name` is what gets submitted/collected by FormData; `id` is what `<label for>`, CSS, and JS-by-id targeting use. They serve different purposes even though both are commonly present on the same input.

## Reading values

```js
formData.get('email');       // single value
formData.getAll('hobbies');  // array, for multi-value fields (e.g. checkboxes sharing a name)
```

## Common pattern: intercept submit, send via fetch

```html
<form id="signup">
  <input name="email" type="email">
  <input name="password" type="password">
  <button type="submit">Sign up</button>
</form>
```
```js
document.querySelector('#signup').addEventListener('submit', async (e) => {
  e.preventDefault(); // stop native full-page navigation/reload
  const formData = new FormData(e.target);

  const response = await fetch('/api/signup', {
    method: 'POST',
    body: formData,
  });
});
```

**Gotcha to remember:** when passing `FormData` directly as `fetch`'s `body`, the browser automatically sets the correct `Content-Type: multipart/form-data` header with the proper boundary string. **Never manually set that header** — it needs the auto-generated boundary value to work correctly; setting it yourself breaks the request.

## Strongest use case: file uploads

FormData correctly handles `<input type="file">` values (actual File objects) alongside regular text fields in one request, without manually constructing multipart encoding:
```js
const formData = new FormData(form);
formData.append('extraField', 'some value'); // can add extra fields too
```

## Modifying entries

```js
formData.append('tag', 'urgent');          // add
formData.set('email', 'new@example.com');  // overwrite existing
formData.delete('tag');                     // remove
formData.has('email');                      // boolean check
```

## Relevance in modern SPA frameworks

Common alternative to manually binding every field to component state (`v-model`/`useState`) for simpler forms — grab the whole form's data in one shot on submit, rather than tracking every keystroke when live reactivity on those fields isn't actually needed.

## Where we left off

FormData API covered in full: construction from a form element, reading/modifying entries, the fetch integration pattern (including the Content-Type header gotcha), and its strongest use case (file uploads alongside text fields). No exercise built yet for this slice.

**Remaining Section 7 slice (last one):** Autocomplete/`autocomplete` token values in detail — this will close out Section 7 (Forms) entirely.
