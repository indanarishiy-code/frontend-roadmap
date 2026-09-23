# Session Summary: Operators, Truthy/Falsy, Template Literals (closes JS Section 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Nullish coalescing (`??`)

```js
const value = input ?? "default";
```
Returns the right-hand side only if the left-hand side is `null` or `undefined` — nothing else. Key distinction from `||`, and the actual reason `??` was added:

```js
const count = 0;
const displayCount = count || 10;  // 10 — WRONG, 0 is falsy so || overrides it
const displayCount2 = count ?? 10;  // 0 — correct, 0 is neither null nor undefined
```
Before `??`, any falsy-but-valid value (`0`, `""`, `false`) would get incorrectly overridden by `||`-based defaults — a genuinely common real bug class. `??` only cares about the specific "missing" states, almost always what's actually meant by a default-value fallback.

**Gotcha:** `??` cannot mix with `||`/`&&` in the same expression without parentheses:
```js
const x = a || b ?? c;   // SyntaxError — ambiguous precedence, JS refuses to guess
const x = (a || b) ?? c;  // must be explicit
```
A deliberate spec decision to force disambiguation rather than pick a potentially surprising default precedence.

## Optional chaining (`?.`)

```js
const city = user?.address?.city;
```
Short-circuits to `undefined` the moment any link in the chain is `null`/`undefined`, instead of throwing a `TypeError`.

```js
// Before: const city = user && user.address && user.address.city;
// After:  const city = user?.address?.city;
```
Replaced a genuinely common, verbose defensive-coding pattern (chains of `&&`) that was everywhere pre-2020.

**Also works on function calls and array/bracket access:**
```js
user?.getProfile?.();  // only calls if it exists, otherwise undefined
arr?.[0];                // safe array access
```
The function-call form (`?.()`) is often forgotten but genuinely useful — calling an optional callback prop without a separate `typeof fn === "function"` check.

**Common combined pattern, worth having as one instinctive unit:**
```js
const city = user?.address?.city ?? "Unknown";
```
Safely walk a potentially-missing chain, then supply a default only if the final result was actually missing.

## Other operators — quick pass

```js
const label = isActive ? "Active" : "Inactive"; // ternary
isVisible && renderSomething();                    // logical AND for conditional rendering (common in JSX/Vue templates)
5 & 3; 5 | 3; 5 ^ 3; ~5; 5 << 1;                    // bitwise — AND/OR/XOR/NOT/shift
```
Bitwise operators are genuinely rare in day-to-day dashboard/CMS frontend work — worth recognizing (permission flags/bitmasks, performance-sensitive number tricks) but not commonly written.

## Truthy/falsy values — the complete, exact list

```js
// The ENTIRE list of falsy values:
false
0
-0
0n        // BigInt zero
""         // empty string
null
undefined
NaN

// Everything else is truthy, including:
"0"        // non-empty string
[]          // empty array
{}          // empty object
"false"    // non-empty string
```
**The two that trip people up most:** `[]` and `{}` are both truthy — an empty array/object still "exists," even though it feels conceptually empty. This is exactly why `if (someArray)` doesn't check for emptiness — need `someArray.length > 0` specifically, since the array itself is always truthy regardless of contents.

## Template literals and tagged templates

```js
const greeting = `Hello, ${name}!`;
const multiline = `Line 1
Line 2`; // literal newlines preserved, no \n needed
```

**Tagged templates — more advanced, genuinely less known:**
```js
function highlight(strings, ...values) { /* strings = literal pieces array, values = interpolated values, given separately */ }
const output = highlight`Name: ${name}, Age: ${age}`;
```
A tagged template lets a function intercept and process a template literal before it becomes a final string, receiving literal text pieces and interpolated values separately rather than pre-concatenated.

**Real-world uses already likely encountered:** styled-components' `` styled.button`...` `` is a tagged template (intercepts the literal to process into actual styles). Also used by `graphql-tag` for parsing GraphQL query strings, and is the mechanism behind native `String.raw` (covered in a later JS section).

## Where we left off

This closes out **JS Section 1: Language Fundamentals** entirely — var/let/const, hoisting, TDZ, closures (with a deep dive), data types/copy semantics, `typeof null` history, coercion, `==` vs `===` (with a deep dive), `??`/`?.` operators, truthy/falsy values, and template literals/tagged templates, all covered across multiple sessions at senior-frontend depth.

Next up per the JavaScript reference doc: **Section 2 — Functions** (function declarations vs expressions vs arrow functions, `this` binding rules, `call()`/`apply()`/`bind()`, default/rest parameters, spread syntax, closures — already covered in depth — IIFEs, higher-order functions, pure functions vs side effects, recursion).
