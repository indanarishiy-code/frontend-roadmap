# Session Summary: Validation Attributes & Constraint Validation API (Section 7, Part 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Validation attributes — the declarative layer

Attributes on `<input>`/`<textarea>` that trigger the browser's built-in validation, no JS required.

```html
<input type="text" required>
<input type="text" minlength="3" maxlength="20">
<input type="number" min="1" max="100" step="5">
<input type="text" pattern="[A-Za-z]{3,}" title="At least 3 letters, no numbers">
```

- **`required`** — boolean attribute (presence = true). Form won't submit and browser shows a native validation bubble if empty.
- **`minlength`/`maxlength`** — character count bounds. `maxlength` actively prevents typing beyond the limit; `minlength` only validates on submit (can type fewer, blocks submission).
- **`min`/`max`/`step`** — for numeric/date-like inputs. `step` defines allowed increments (e.g. `min="1" max="100" step="5"` only allows 1, 6, 11...100); non-step-aligned values fail validation.
- **`pattern`** — a regex the value must match.
  - **Implicitly anchored** — browser treats it as wrapped in `^(?:...)$`, matching the entire value, not a substring
  - **Always pair with `title`** — this is what the browser shows in the validation error message explaining the expected format. Without it, users get a generic "please match the requested format" with no clue what format is actually needed.

**Key concept:** these attributes are declarative *triggers*, not the validation system itself — they set constraints checked by the browser's underlying validation engine (the Constraint Validation API).

## Constraint Validation API — the JS interface

The programmatic layer for interacting with browser validation: checking validity, finding out *why* something's invalid, customizing error messages, triggering validation manually.

### `element.checkValidity()`
Returns `true`/`false` — checks against all constraints without submitting the form.
```js
if (!input.checkValidity()) { console.log('Invalid!'); }
```

### `element.reportValidity()`
Same check, but also shows the native browser validation bubble UI (same as a failed submit) — useful for triggering native feedback at a custom moment (e.g. on blur).

### `ValidityState` — the real depth
Every validatable element has a `.validity` property with boolean flags for exactly which constraint failed:
```js
input.validity
// { valueMissing, typeMismatch, patternMismatch, tooShort, tooLong,
//   rangeUnderflow, rangeOverflow, stepMismatch, valid, ... }
```
This is what `checkValidity()` alone doesn't give you — a boolean says *that* it failed, `ValidityState` says *why*, enabling specific helpful messages instead of generic ones.

### `setCustomValidity()` — custom validation messages
```js
input.addEventListener('input', () => {
  if (input.value.includes(' ')) {
    input.setCustomValidity('Username cannot contain spaces');
  } else {
    input.setCustomValidity(''); // must clear it or field stays permanently invalid
  }
});
```
**Critical gotcha:** once set with a non-empty string, the field is invalid regardless of anything else, and stays that way until explicitly cleared with `setCustomValidity('')`. Forgetting to clear it is a real, common bug — field looks fine and passes every declarative constraint but silently won't submit due to a stale custom message.

### Practical combined pattern
```js
form.addEventListener('submit', (e) => {
  const input = document.querySelector('#age');
  if (input.validity.rangeUnderflow) {
    input.setCustomValidity('You must be at least 18.');
    input.reportValidity();
    e.preventDefault();
  } else {
    input.setCustomValidity('');
  }
});
```

## Is this "the base of all form components in a library"? — nuanced answer

**Partially, but mostly no** for production form libraries.

**Some libraries do lean on it:** VeeValidate and other lightweight libraries build on native constraint attributes/`ValidityState`, especially for basic HTML5-native validation modes.

**Most heavier-duty libraries (React Hook Form, Formik) bypass it entirely, reimplementing validation in JS**, because the native API can't handle:
- **Cross-field validation** — e.g. "password confirmation must match password" — native API has no concept of one field's validity depending on another
- **Schema-based validation** — Zod/Yup-style schemas defined in JS/TS scale much better for complex, nested form state than HTML attributes
- **Consistent cross-browser UI** — native validation bubbles look different per browser and are hard to style consistently; most design systems suppress native UI (`novalidate`) and build fully custom error display
- **Async validation** — e.g. "is this username taken" via API call — no native concept of this

**The actual relationship:** the Constraint Validation API is the foundation for simple, native-only validation (good for small forms/prototypes), but most production form libraries reimplement validation in JS for the more complex real-world cases. What's still valuable: understanding `required`/`pattern`/`ValidityState` gives the conceptual vocabulary (what "valid," "required," "pattern mismatch" mean) that these libraries reimplement in their own way — not wasted knowledge, just not literally what powers most libraries internally.

## Where we left off

Validation attributes and the Constraint Validation API — the topic that originally prompted this whole study series — covered in full, including the practical nuance about how it relates (or doesn't) to real-world form libraries like React Hook Form. No exercise built yet for this slice.

**Remaining Section 7 slices (planned for follow-up sessions):**
- FormData API
- Autocomplete/`autocomplete` token values in detail
