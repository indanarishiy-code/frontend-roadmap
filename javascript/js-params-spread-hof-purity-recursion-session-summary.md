# Session Summary: Parameters, Spread, IIFEs, Higher-Order Functions, Purity, Recursion (closes JS Section 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Default parameters

```js
function greet(name = "Guest") { return `Hello, ${name}`; }
greet();          // "Hello, Guest"
greet(undefined);  // "Hello, Guest" — undefined explicitly triggers the default too
greet(null);        // "Hello, null" — null does NOT trigger the default
```
**Key gotcha:** defaults only kick in for `undefined` specifically (omitted argument or explicit `undefined`) — not for `null`, `0`, or `""`. Behaves like `??` internally, not `||`, connecting directly to the earlier `??` vs `||` distinction.

**Defaults can reference earlier parameters:**
```js
function createUser(name, greeting = `Hello, ${name}`) { return greeting; }
```
Evaluated left to right — a later default can use an earlier parameter's value, not vice versa.

## Rest parameters

```js
function sum(...numbers) { return numbers.reduce((total, n) => total + n, 0); }
sum(1, 2, 3); // 6
```
Collects remaining arguments into a real array. Must be the LAST parameter — `function foo(...rest, last)` is a syntax error. Can coexist with named parameters before it: `function logFirst(first, ...others) {}`.

## Spread syntax — the same `...`, opposite direction from rest

```js
// Rest: gathers values INTO an array (function parameters/destructuring)
function sum(...numbers) { }
// Spread: expands an array OUT into individual values (function calls/array/object literals)
sum(...[1, 2, 3]); // equivalent to sum(1, 2, 3)
```

**Array spread:**
```js
const arr2 = [...arr1, 4, 5]; // NEW array, arr1 untouched — replaces the older arr1.concat([4, 5])
```

**Object spread — immutable update pattern:**
```js
const updatedUser = { ...user, age: 31 }; // NEW object
```

**Critical nuance — spread is only a SHALLOW copy:**
```js
const state = { user: { name: "Alex" }, count: 0 };
const newState = { ...state, count: 1 };
newState.user.name = "Bob";
console.log(state.user.name); // "Bob" — nested object is STILL SHARED, not actually copied
```
Top-level keys get new references; any nested object/array inside is the exact same reference as before. A genuinely common source of bugs in state management (Pinia included) when someone assumes spread gives a full deep copy and mutates a nested value, accidentally affecting the "old" state too.

## IIFEs (Immediately Invoked Function Expressions)

```js
(function() {
  const privateVar = "hidden";
  console.log("runs immediately");
})();
```
Wraps a function in parentheses (making it an expression, since declarations can't be immediately invoked this way) then calls it immediately.

**Why it existed:** before ES modules and `let`/`const` block scoping, this was the only way to create a private, isolated scope avoiding global namespace pollution.

**Current relevance — genuinely reduced:** ES modules already give each file its own private scope; `let`/`const` already give block-scoping without a function wrapper. Still seen in older codebases, some bundler output, or self-invoking async setup patterns — no longer a go-to for everyday scoping.

## Higher-order functions

A function that accepts another function as an argument, returns a function, or both.
```js
[1, 2, 3].map(n => n * 2);            // accepts a function
function multiplier(factor) {           // returns a function
  return (n) => n * factor;
}
```
Array methods (`map`, `filter`, `reduce`) are the higher-order functions used constantly — worth recognizing them explicitly as higher-order functions, since that's the actual CS term used in interview definitions.

## Pure functions vs side effects

```js
function add(a, b) { return a + b; }        // pure — same input, same output, touches nothing external
let total = 0;
function addToTotal(n) { total += n; }        // impure — mutates external state
```
A pure function's output depends only on inputs, with no mutation of arguments/external variables, no DOM manipulation, no network calls.

**Why this matters practically:** pure functions are trivially testable (same input/output, no setup/teardown), safely reusable, predictable under concurrent/async execution. Directly why Vuex/Pinia getters and computed properties are expected to be pure — a computed property with side effects breaks the reactivity system's ability to reliably cache/recompute it.

## Recursion

```js
function factorial(n) {
  if (n <= 1) return 1;         // base case — without this, infinite recursion
  return n * factorial(n - 1);  // recursive case
}
```
Every recursive function needs a base case (stops recursion) and a recursive case (moves toward the base case).

**Senior practical note:** JavaScript does not have reliable tail-call optimization in most engines (despite being technically part of the ES2015 spec) — deeply recursive functions on large inputs can genuinely hit a stack overflow in real JS engines. This is why iterative solutions (loops) are often preferred over recursion for large-dataset processing in production JavaScript, unlike languages where tail-call optimization is dependable.

## Where we left off

This closes out **JS Section 2: Functions** entirely — declarations vs expressions, `this` binding (deep dive in prior session), `call`/`apply`/`bind` (deep dive in prior session), default/rest parameters, spread syntax (with the shallow-copy gotcha directly relevant to Pinia state management), IIFEs, higher-order functions, pure functions vs side effects, and recursion. Closures already covered in Section 1.

Next up per the JavaScript reference doc: **Section 3 — Objects** (object literals, property shorthand, computed property names, dot vs bracket access, `Object.keys()`/`values()`/`entries()`/`assign()`/`freeze()`/`seal()`, `Object.groupBy()` (ES2024), getters/setters, property descriptors, prototypes and the prototype chain, `Object.create()`).
