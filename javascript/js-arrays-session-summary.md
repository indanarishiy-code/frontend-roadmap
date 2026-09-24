# Session Summary: Arrays (JS Section 5)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Array creation

```js
Array.from({ length: 5 }, (_, i) => i * 2); // [0, 2, 4, 6, 8]
Array.from("hello");                          // ["h", "e", "l", "l", "o"] — strings are iterable
Array.of(7);                                    // [7]
Array.isArray([1, 2, 3]);                       // true — reliable check, since typeof arrays is "object"
```
**Genuine gotcha:** `new Array(7)` creates an empty array with `length: 7` (sparse, no real elements), while `Array.of(7)` creates `[7]`, a single-element array. This inconsistency in the `Array` constructor's behavior based on argument count is exactly why `Array.of()` exists — predictable behavior regardless of what's passed.

## Mutating vs non-mutating — the distinction that matters most

```js
// MUTATING — changes original in place
arr.push(4); arr.pop(); arr.sort(); arr.reverse(); arr.splice(1, 1);

// NON-MUTATING — returns a new array
const mapped = arr.map(n => n * 2);
const filtered = arr.filter(n => n > 1);
```
**Genuinely important:** `sort()` and `reverse()` mutate — a common real source of bugs, especially in frameworks with reactivity. If a Vue reactive array gets `.sort()`-ed directly, it mutates the actual reactive state in place — usually still triggers Vue 3's Proxy-based reactivity correctly, but can cause subtle issues with change tracking, undo/redo logic, or anywhere the original array order was assumed preserved elsewhere.

## ES2023 immutable array methods — the direct fix

```js
const arr = [3, 1, 2];
const sorted = arr.toSorted();           // [1, 2, 3] — NEW array
const reversed = arr.toReversed();        // [2, 1, 3] — NEW array
const spliced = arr.toSpliced(1, 1, 99);  // NEW array with splice applied
const updated = arr.with(0, 100);          // [100, 1, 2] — NEW array, index 0 replaced
// arr itself: [3, 1, 2] — completely unchanged after all of the above
```
**Why this matters:** before these existed, a non-mutating sort required `[...arr].sort()` or `arr.slice().sort()` — copy first, then mutate the copy. These methods are direct one-line replacements. The `to` + verb naming convention is a deliberate signal: any method starting with `to` returns a new array rather than mutating.

**Direct relevance to Pinia/Vuex state updates:** immutable array updates are exactly the pattern state management wants (change detection, undo/redo, avoiding accidental shared-reference bugs):
```js
// Old pattern
this.items = [...this.items].sort((a, b) => a.value - b.value);
// New pattern
this.items = this.items.toSorted((a, b) => a.value - b.value);
```

**`with()` specifically** replaces the awkward manual slice-and-spread pattern for immutably replacing one element:
```js
// Old: const updated = [...arr.slice(0, 1), "X", ...arr.slice(2)];
// New: const updated = arr.with(1, "X");
```

## Destructuring arrays

```js
const [first, second, ...rest] = [1, 2, 3, 4, 5]; // first=1, second=2, rest=[3,4,5]
const [, , third] = [1, 2, 3]; // skipping elements with empty slots — third = 3
```
Skipped-slot destructuring (`[, , third]`) worth recognizing when seen — the empty commas can look confusing at first glance.

## Array-like objects vs true arrays

```js
function example() {
  console.log(arguments);              // array-LIKE — has length/indices, no map/filter/etc.
  const realArray = Array.from(arguments); // convert to a real array
}
document.querySelectorAll('div'); // NodeList — also array-like
```
**Practical distinction:** array-like objects have `length` and numeric indices but don't inherit from `Array.prototype`, so `map`/`filter`/`reduce` aren't directly available. `Array.from()` is the standard conversion tool — turns any iterable or array-like object into a genuine array supporting the full array method set.

## `Array.fromAsync` — ES2026, cutting-edge

```js
async function* generateNumbers() { yield 1; yield 2; yield 3; }
const arr = await Array.fromAsync(generateNumbers()); // [1, 2, 3]
```
**Problem solved:** `Array.from()` works with synchronous iterables but has no clean way to handle an async iterable (something yielding promises, like an async generator) — previously required manually looping with `for await...of` and pushing into an array. `Array.fromAsync()` collects natively in one line. Very recent — know it exists conceptually, not yet assumed broadly supported without checking current Baseline status.

## Where we left off

Section 5 (Arrays) covered in full — the immutable array methods (`toSorted`/`toReversed`/`toSpliced`/`with`) and their direct relevance to Pinia state updates flagged as the most currently practical, load-bearing piece given the actual Vue/Pinia work context. `Array.fromAsync` covered as a cutting-edge item.

Next up per the JavaScript reference doc: **Section 6 — Iteration & Iterables** (`for`/`for...in`/`for...of`, iterables and the iterator protocol via `Symbol.iterator`, generators, Iterator Helpers — a flagged recent-gap topic — async iterators and `for await...of`).
