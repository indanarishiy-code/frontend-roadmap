# Session Summary: Objects — Literals, Methods, Descriptors (JS Section 3, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Prototypes/prototype chain/Object.create() are planned for a dedicated follow-up session.*

## Object literals, shorthand, computed properties

```js
const name = "Alex";
const age = 30;
const user = { name, age };        // property shorthand — equivalent to { name: name, age: age }
const user2 = { [key]: "admin" };   // computed property name — key from a variable/expression
```
Shorthand is everywhere in modern code — treat as second nature. Computed properties let a key come from a variable/expression rather than a hardcoded string — common when building objects dynamically from data.

## Dot vs bracket notation

```js
user.name;         // dot — key must be a valid identifier, known at write-time
user["name"];        // bracket — key can be any expression, including dynamic
user["first-name"];  // dot notation CAN'T do this — hyphens aren't valid identifiers
```
Bracket notation required whenever the key isn't a simple known identifier — dynamic keys, special characters, runtime-computed keys.

## `Object.keys()` / `values()` / `entries()`

```js
Object.keys(obj);    // ["a", "b"]
Object.values(obj);   // [1, 2]
Object.entries(obj);  // [["a", 1], ["b", 2]]
for (const [key, value] of Object.entries(obj)) { }
```
`Object.entries()` + destructuring in `for...of` is the modern, idiomatic iteration pattern, replacing `for...in` (which has real gotchas around inherited properties).

## `Object.assign()` vs spread

```js
const merged = Object.assign({}, obj1, obj2);
const merged2 = { ...obj1, ...obj2 }; // same result, more modern syntax
```
Both merge shallowly, later sources overwrite matching keys. Spread has largely replaced `Object.assign()` for this use case, but `Object.assign()` retains one distinct capability: **mutating an existing object in place** if a real object (not `{}`) is passed as the first argument:
```js
Object.assign(obj1, obj2); // MUTATES obj1 directly, no new object created
```

## `Object.freeze()` vs `Object.seal()` — precise distinction

```js
const frozen = Object.freeze({ a: 1 });
frozen.a = 2; frozen.b = 3; delete frozen.a; // all fail

const sealed = Object.seal({ a: 1 });
sealed.a = 2;   // WORKS — existing property values still modifiable
sealed.b = 3;    // fails — can't add new properties
delete sealed.a; // fails — can't delete properties
```
`seal()` prevents adding/removing properties, but allows modifying existing values. `freeze()` does everything `seal()` does, plus prevents modifying existing values too — the stricter of the two.

**Gotcha — both are only shallow:**
```js
const frozen = Object.freeze({ nested: { a: 1 } });
frozen.nested.a = 99; // WORKS — the nested object was never frozen itself
```
Same shallow-copy gotcha as spread syntax, applied to freezing. **Unified mental model:** most built-in object operations (spread, `Object.assign`, `Object.freeze`) operate only one level deep, never recursively, unless explicitly using something like `structuredClone()` or a deep-freeze utility.

## `Object.groupBy()` — ES2024

```js
const items = [
  { type: "fruit", name: "apple" },
  { type: "vegetable", name: "carrot" },
  { type: "fruit", name: "banana" },
];
const grouped = Object.groupBy(items, item => item.type);
// { fruit: [apple, banana], vegetable: [carrot] }
```
**Problem solved:** before this, grouping an array by a computed key required a manual `reduce()` with accumulator logic, or pulling in Lodash's `groupBy()`. Now a native one-line replacement — reduces a small but real Lodash dependency for many projects.

## Getters and setters

```js
const user = {
  firstName: "Alex", lastName: "Smith",
  get fullName() { return `${this.firstName} ${this.lastName}`; },
  set fullName(value) { [this.firstName, this.lastName] = value.split(" "); }
};
user.fullName;              // "Alex Smith" — called like a property, not a method
user.fullName = "Jordan Lee"; // triggers the setter
```
Lets a property look like a plain value externally while running computed logic underneath — the same underlying concept Vue's `computed` properties are built on, at the raw JS object level.

## Property descriptors and `Object.defineProperty()`

```js
Object.defineProperty(obj, 'x', {
  value: 42,
  writable: false,    // can't be reassigned
  enumerable: false,   // won't show in Object.keys()/for...in
  configurable: false   // can't be deleted or redefined
});
```
The low-level mechanism getters/setters, `freeze()`, and `seal()` are all built on. Every property has these hidden descriptor flags — normal assignment (`obj.x = 42`) uses sensible defaults (`writable`/`enumerable`/`configurable` all `true`).

**Why this matters at senior level:** this is genuinely how Vue 2's entire reactivity system worked internally — used `Object.defineProperty()` to intercept every property's get/set, which is exactly why Vue 2 couldn't detect newly added properties or array index changes (interception must be set up per-property in advance). Vue 3 switched to `Proxy` specifically to solve this limitation.

## Where we left off

First slice of JS Section 3 covered: object literals/shorthand/computed properties, dot vs bracket notation, the key `Object.*` utility methods (with precise `assign` vs spread and `freeze` vs `seal` distinctions), `Object.groupBy()` as a Lodash-replacement, and getters/setters/property descriptors — including the direct connection to Vue 2's `defineProperty`-based reactivity vs Vue 3's `Proxy` approach. Remaining Section 3: prototypes and the prototype chain, `Object.create()` — planned as a dedicated deep-dive session given their complexity.
