# Session Summary: Numbers, Math, BigInt + isNaN/isFinite Deep Dive + Math.random() Security (JS Section 8)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Floating point precision — the famous gotcha

```js
0.1 + 0.2; // 0.30000000000000004
0.1 + 0.2 === 0.3; // false
```
**Why:** JS numbers are IEEE 754 double-precision floats, representing most decimal fractions as binary approximations, not exact values (the same way 1/3 can't be exact in decimal). `0.1`/`0.2` are each already tiny approximations before adding; the error compounds into something visible.

**Practical fix:**
```js
(0.1 + 0.2).toFixed(2);              // "0.30" — rounds for display, returns a STRING
Math.round((0.1 + 0.2) * 100) / 100;  // 0.3 — rounds for numeric comparison, returns a NUMBER
Number((0.1 + 0.2).toFixed(2));        // 0.3 — combine both if a number is needed back
```
`toFixed()` returns a string — remember to convert back with `Number()`/unary `+` if more math is needed.

**Real relevance for currency/financial calculations:** summing prices, totals, percentage discounts are exactly where this bites. Safest pattern: work in integer cents internally (multiply by 100, do integer math, divide back only for display) rather than floating-point math directly on dollar amounts — integers don't have this approximation problem.

## `parseInt`/`parseFloat` vs `Number()` — why partial parsing exists at all

```js
parseInt("42px");   // 42 — reads valid digits, stops at first non-numeric character
Number("42px");        // NaN — requires the ENTIRE string to be numeric, no partial parsing
```

**Why `parseInt`/`parseFloat` don't just return `NaN` immediately:** they were deliberately designed to extract a number embedded in a larger string — reading digits left to right until hitting something that can't continue, then returning what was read so far. The actual historical use case: parsing CSS pixel values, HTML attribute values, and other strings that legitimately mix a number with a unit/label (`parseInt(element.style.width)` on `"300px"` is a genuinely common real pattern). Returning `NaN` immediately would defeat the entire purpose — `Number()` already exists for strict whole-string validation.

**Verified (MDN): `Number.parseInt()`/`Number.parseFloat()` are identical to their global counterparts** — no behavior difference, purely relocated onto `Number` for namespace consistency (ES2015). Safe to use either form.

## `isNaN()`/`isFinite()` vs `Number.isNaN()`/`Number.isFinite()` — genuine coercion difference, verified via MDN

**Why the two "same-named" functions behave differently — they are NOT the same function:**
```js
isNaN;             // the GLOBAL function
Number.isNaN;        // a DIFFERENT function, added later as a static method on Number
isNaN === Number.isNaN; // false — genuinely separate function objects
```
The global `isNaN()`/`isFinite()` predate `Number` having static utility methods (1995-era). ES2015 added stricter, non-coercing versions on `Number` as improved replacements, without changing the originals (to avoid breaking existing code relying on the old coercing behavior) — both were kept permanently.

```js
isNaN("hello");          // true — coerces "hello" to NaN first, THEN checks — misleading
Number.isNaN("hello");    // false — no coercion; "hello" isn't literally the NaN value

isFinite("0");             // true — "0" coerced to 0, which IS finite
Number.isFinite("0");       // false — "0" is a string, not a number, no coercion
```
MDN confirms directly: "Because coercion inside the isNaN() function can be surprising, you may prefer to use Number.isNaN()" — identical wording pattern for `isFinite()`.

**Clean summary table:**
| Global function | `Number.*` equivalent | Behaves differently? |
|---|---|---|
| `isNaN()` | `Number.isNaN()` | **Yes** — global coerces first |
| `isFinite()` | `Number.isFinite()` | **Yes** — global coerces first |
| `parseInt()` | `Number.parseInt()` | No — identical, just relocated |
| `parseFloat()` | `Number.parseFloat()` | No — identical, just relocated |

**Practical rule:** always use `Number.isNaN()`/`Number.isFinite()` for correctness (avoids the coercion trap). `parseInt`/`Number.parseInt` and `parseFloat`/`Number.parseFloat` are interchangeable — no correctness risk either way, just a style/namespace preference some linters enforce.

## SonarQube rule `S7773` — Number statics preferred over global equivalents

The linter rule bundles two different kinds of warning together:
- `parseInt`/`parseFloat` → `Number.parseInt`/`Number.parseFloat`: pure namespace hygiene, zero behavior difference, auto-fixable
- `isNaN`/`isFinite` → `Number.isNaN`/`Number.isFinite`: genuine correctness fix, explicitly noted by the rule as having "slightly different behavior"

## `new Number()` / `new String()` / `new Boolean()` — a separate, real pitfall (ESLint `no-new-wrappers`)

```js
const num = Number(33);        // FINE — converts to a primitive number
const numObject = new Number(33); // FLAGGED — creates a wrapper OBJECT, not a primitive
typeof numObject; // "object", NOT "number"

const bool = new Boolean(false);
if (bool) { /* this runs! */ } // every OBJECT is truthy, even one wrapping false — a real silent logic bug
```
**The distinction:** `Number()`/`String()`/`Boolean()` called as plain functions (no `new`) for type conversion are completely correct and standard. Calling them with `new` creates a wrapper object that no longer behaves like the primitive, causing `typeof` mismatches and the "objects are always truthy" trap. This is a separate rule/pitfall from the `isNaN` coercion issue, easy to conflate given the shared `Number` name.

**Related TypeScript rule (`@typescript-eslint/no-wrapper-object-types`):** flags the capitalized wrapper *types* (`Number`/`String`/`Boolean`) in type annotations — use lowercase primitive types (`number`/`string`/`boolean`) instead. A different, type-annotation-level version of the same underlying wrapper-vs-primitive distinction.

## `Number.isNaN`/`isInteger` and other Number utility methods

```js
Number.isInteger(5);    // true
Number.isInteger(5.0);   // true — 5.0 IS an integer value in JS, no separate float type
Number.isInteger(5.5);   // false
```

## Math object — common methods

```js
Math.round(4.5); Math.floor(4.9); Math.ceil(4.1); Math.abs(-5);
Math.max(1, 5, 3); Math.min(1, 5, 3);
Math.random(); // 0 (inclusive) to 1 (exclusive)
Math.pow(2, 10); // equivalent to 2 ** 10
```
**Gotcha:** `Math.max()`/`Math.min()` take individual arguments, not an array:
```js
Math.max([1, 5, 3]);    // NaN — wrong
Math.max(...[1, 5, 3]); // 5 — spread into individual arguments first
```

## `Math.random()` — real security vulnerability, verified via search

**Why it's insecure:** `Math.random()` is a PRNG (pseudo-random number generator) — deterministic, computed from an internal state via a fixed algorithm (e.g. xorshift128+), optimized for speed and statistical distribution, not security. Given enough observed outputs, an attacker can work backward to determine internal state and predict all future values — unpredictable to casual observation, not to someone doing the math.

**Why this is a real, current vulnerability class, not theoretical:** two CVEs published within 24 months confirm real-world exploitation — CVE-2024-40762 (weak PRNG in an authentication token generator → authentication bypass) and CVE-2025-22150 (undici, Node's HTTP client, used `Math.random()` for multipart/form-data boundaries → predictable API request boundaries). AI coding tools frequently generate `Math.random()` for password reset tokens/session IDs/invitation codes, making those guessable.

**The fix:**
```js
// INSECURE — never for tokens, sessions, passwords, CSRF tokens
const token = Math.random().toString(36);

// SECURE — cryptographically random
crypto.randomUUID();          // simplest, built-in
crypto.getRandomValues(array); // lower-level, more control
```
`crypto.getRandomValues()`/`crypto.randomUUID()` pull from the OS's actual cryptographically secure random source, designed specifically to resist prediction attacks.

**Practical rule:** `Math.random()` remains fine for non-security use (shuffling display order, random UI variant, demo/sample data). Any security-relevant use (tokens, unguessable IDs, anything tied to auth) requires `crypto.randomUUID()`/`crypto.getRandomValues()` instead — almost certainly what any security scanner near token/ID/session-generation code is flagging.

## BigInt — when and why

```js
const big = 9007199254740993n; // 'n' suffix marks a BigInt literal
Number.MAX_SAFE_INTEGER;         // 9007199254740991 — largest integer JS represents EXACTLY
9007199254740992 + 1;             // WRONG — silently loses precision past this point
9007199254740993n + 1n;            // correct — BigInt has no such limit
5n + 5;                             // TypeError — cannot mix BigInt and Number directly
5n + BigInt(5);                     // 10n — must explicitly convert first
```
**Problem solved:** regular numbers (IEEE 754 doubles) can only represent integers exactly up to `MAX_SAFE_INTEGER` — beyond that, math silently produces wrong results with no error. BigInt is a separate arbitrary-precision integer type, at the cost of no decimal support and no implicit mixing with regular numbers.

**Practical relevance:** genuinely rare in typical dashboard work — relevant for large 64-bit database IDs exceeding `MAX_SAFE_INTEGER`, cryptographic operations, or precise large-integer math. Not needed if business IDs/counts/financial figures stay within safe integer range.

## Typed arrays — `Float16Array`

```js
const arr16 = new Float16Array([1.5, 2.5]); // newer, more compact precision
```
Fixed-precision binary numeric storage (unlike regular JS arrays using full double precision for every number). Useful for memory-constrained numeric work (ML model weights, WebGL/graphics buffers). **Practical relevance for typical dashboard work: quite low** — squarely graphics/ML-adjacent territory, worth recognizing rather than expecting to use.

## Where we left off

Section 8 (Numbers, Math & BigInt) covered, expanded by a genuinely thorough real-world digression: the precise mechanism behind `isNaN()`/`Number.isNaN()` and `isFinite()`/`Number.isFinite()` divergence (verified via MDN), confirmation that `parseInt`/`parseFloat` have no such divergence, the separate `new Number()` wrapper-object pitfall (ESLint `no-new-wrappers`), the SonarQube `S7773` rule's actual scope, and a fully verified (via search, current CVEs) explanation of why `Math.random()` is a real security vulnerability and what to use instead (`crypto.randomUUID()`/`crypto.getRandomValues()`).

Next up per the JavaScript reference doc: **Section 9 — Dates & Time** (legacy Date object and its pitfalls, the Temporal API as the modern replacement — flagged as a recent-gap topic worth learning fresh — timezone handling considerations).
