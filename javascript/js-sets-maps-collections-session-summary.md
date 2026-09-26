# Session Summary: Sets, Maps & Collections (JS Section 15)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `Set` — genuinely unique values

```js
const ids = new Set([1, 2, 2, 3, 3, 3]); // Set(3) {1, 2, 3} — duplicates auto-removed
ids.add(4); ids.has(2); ids.delete(1);
ids.size;   // not .length — worth remembering this difference from arrays
[...ids];    // convert to array when array methods are needed
```
**Problem solved cleanly:** deduplicating an array used to require `array.filter((v, i) => array.indexOf(v) === i)` or a manual lookup object. `[...new Set(array)]` is now the standard, idiomatic one-liner for deduplication.

## `Map` — genuinely better than plain objects for certain cases

```js
const userRoles = new Map();
userRoles.set('alex', 'admin');
userRoles.get('alex');   // "admin"
userRoles.size;             // real property
for (const [key, value] of userRoles) { }
```

**Why `Map` genuinely beats a plain object, not just style:**
- **Any value can be a key** — a plain object silently coerces keys to strings (`obj[{}]` becomes `obj["[object Object]"]`); a `Map` can use an actual object, function, or DOM element as a genuinely distinct key
- **Guaranteed insertion-order iteration** — plain objects have spec-defined ordering quirks (integer-like keys sort numerically first, breaking insertion order); `Map` always iterates in the exact order entries were added
- **`size` is a real property**, not `Object.keys(obj).length`
- **No prototype pollution risk** — plain object keys can collide with inherited properties (`obj.toString`, `obj.constructor`); `Map` keys are isolated from any prototype chain

**Practical relevance:** for a lookup table keyed by something other than a simple string/number (a component reference, a DOM node, object identity), `Map` is the correct tool — plain objects genuinely can't do this correctly.

## `WeakSet`/`WeakMap` — direct connection to the memory-leak session (Section 13)

```js
const cache = new WeakMap();
function processElement(el) {
  if (cache.has(el)) return cache.get(el);
  const result = expensiveComputation(el);
  cache.set(el, result);
  return result;
}
```
`WeakMap` keys must be objects (not primitives), and are held **weakly** — if the only remaining reference to an object is as a `WeakMap` key, the garbage collector can still reclaim it. A normal `Map` would keep that object alive forever just by having it as a key, even after every other part of the code stopped using it — a genuine, real memory leak source for exactly this kind of DOM-element-keyed caching. This is precisely why `WeakMap`/`WeakSet` exist as separate types, and why using them for caches tied to DOM elements or component instances is a real recommended practice, not a micro-optimization.

**The tradeoff:** `WeakMap`/`WeakSet` are **not iterable** — no `.size`, no `for...of`, no listing all entries. Deliberate design constraint, not an oversight — since entries can silently disappear at any moment (whenever GC reclaims a key), iterating "current contents" would be fundamentally unreliable. Use them specifically for "do I have data for this specific object" lookups — never "give me everything currently stored."

## New ES2025 Set methods — native math-like set operations

```js
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);
a.union(b);                // {1, 2, 3, 4} — everything in either
a.intersection(b);          // {2, 3} — only what's in both
a.difference(b);             // {1} — in a but NOT in b
a.symmetricDifference(b);    // {1, 4} — in exactly one, not both
a.isSubsetOf(b);               // false
a.isSupersetOf(b);             // false
a.isDisjointFrom(b);            // false — share NO elements at all?
```
**Why useful, not just math trivia:** before these, computing "items in this selection but not that one" required manually writing `filter()`/`includes()` combinations every time. These make common data-comparison operations (permission set differences, tag overlap, filter comparisons) a single readable method call instead of hand-rolled loop logic.

## `Map.prototype.getOrInsert()`/`getOrInsertComputed()` — ES2026

```js
// OLD — the "check then set" dance
function getCache(key) {
  if (!cache.has(key)) cache.set(key, computeExpensiveValue(key));
  return cache.get(key);
}
// NEW — one line
function getCache(key) {
  return cache.getOrInsertComputed(key, () => computeExpensiveValue(key));
}
```
**Problem solved:** "check if it exists, compute and store if not, return it" is one of the most common `Map`/object patterns for caching/memoization (ties directly to the memoization discussion in Section 13) — this collapses the three-line dance into one atomic, readable call.

## Practical decision — when to reach for Map/Set over plain objects/arrays

- **Plain object** — still completely fine, usually simpler, for genuinely simple key-value data with string keys, no insertion-order requirement
- **`Map`** — reach for it when keys aren't strings, insertion order genuinely matters, or the collection is a dynamic, frequently-added-to/removed-from lookup structure
- **`Set`** — reach for it for uniqueness/deduplication and set-operation needs (the new ES2025 methods)
- **`WeakMap`/`WeakSet`** — reach for these specifically for caches keyed by objects/DOM elements/component instances, where the cache entry should disappear automatically once the underlying object is no longer used elsewhere

## Where we left off

Section 15 (Sets, Maps & Collections) covered in full — `Set`/`Map` basics and their genuine advantages over plain objects/arrays, `WeakMap`/`WeakSet`'s direct connection to the memory-leak prevention topic from Section 13, the new ES2025 Set operation methods, and `Map.prototype.getOrInsert()` as a memoization-pattern shortcut.

Next up per the JavaScript reference doc: **Section 16 — Symbols & Metaprogramming** (Symbol primitive and well-known symbols, `Proxy` and `Reflect`, custom iterator/iterable implementation via `Symbol.iterator` — already touched on in Section 6).
