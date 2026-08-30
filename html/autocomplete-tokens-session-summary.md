# Session Summary: Autocomplete Tokens (Section 7, Part 4 — closes Section 7)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `autocomplete` — beyond just "on"/"off"

Form-level `autocomplete="on"`/`"off"` is a blunt toggle. At the field level, HTML defines specific standardized tokens that tell the browser exactly *what kind* of data a field expects — not just whether to remember it.

```html
<input type="text" name="fname" autocomplete="given-name">
<input type="text" name="lname" autocomplete="family-name">
<input type="email" autocomplete="email">
<input type="tel" autocomplete="tel">
<input type="text" autocomplete="address-line1">
<input type="text" autocomplete="postal-code">
<input type="text" autocomplete="cc-number">
<input type="text" autocomplete="cc-exp">
```

## Why this matters practically

Browsers use these tokens to **auto-fill entire forms** from saved profile/payment data — not just repeat previously typed values. Correctly tokenized checkout forms get fully populated (name, address, card) from saved autofill data in one click. Generic/missing tokens force manual typing even when the browser has the data saved.

**Real business impact:** a genuine, measurable conversion-rate factor for checkout/signup forms.

## Common tokens by category

- **Name:** `name`, `given-name`, `family-name`, `nickname`
- **Contact:** `email`, `tel`, `tel-country-code`
- **Address:** `street-address`, `address-line1`, `address-line2`, `address-level1` (state/province), `address-level2` (city), `postal-code`, `country`
- **Payment:** `cc-name`, `cc-number`, `cc-exp`, `cc-exp-month`, `cc-exp-year`, `cc-csc`
- **Account/auth:** `username`, `new-password`, `current-password`, `one-time-code`

## `new-password` vs `current-password` — important pair

```html
<input type="password" autocomplete="current-password">  <!-- login form -->
<input type="password" autocomplete="new-password">       <!-- signup / change password -->
```
Tells password managers whether to **fill an existing saved password** (`current-password`, login) or **offer to generate/save a new one** (`new-password`, signup/reset). Getting this wrong is a real, common bug — password managers fail to offer autofill on login forms, or fail to offer to save new credentials on signup forms.

## `one-time-code` — modern, genuinely useful

```html
<input type="text" autocomplete="one-time-code" inputmode="numeric">
```
On mobile, lets the browser auto-read a verification code from an incoming SMS and offer to fill it directly — real UX improvement for 2FA/OTP flows, easy to miss if the token isn't known.

## Where we left off

This closes out **Section 7: Forms** entirely, across multiple sessions:
1. `<form>` structure, `<label>`/`<fieldset>`/`<legend>`, full `<input>` type surface, `<textarea>`/`<select>`/`<datalist>`, `<button>` type gotcha, `<output>`/`<progress>`/`<meter>`
2. Validation attributes (`required`, `pattern`, `min`/`max`/`step`, `minlength`/`maxlength`) and the Constraint Validation API (`checkValidity()`, `reportValidity()`, `ValidityState`, `setCustomValidity()`) — including the nuanced reality of how (or whether) real form libraries like React Hook Form build on this
3. FormData API — construction, reading/modifying entries, fetch integration, file upload handling
4. Autocomplete tokens — field-level tokens for browser autofill, `new-password`/`current-password` distinction, `one-time-code`

Next up per the HTML reference doc: **Section 8 — Interactive Elements** (`<details>`/`<summary>`, `<dialog>`, PopoverAPI, Invoker Commands, `<template>`).
