# Session Summary: var/let/const, Hoisting, TDZ, Closures (JS Section 1, Part 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## var vs let vs const

`var` is function-scoped (or global if declared outside a function). `let`/`const` are block-scoped — only exist within the nearest `{}`, including `if`/`for`/bare blocks.

```js
if (true) {
  var a = 1;
  let b = 2;
}
console.log(a); // 1 — var leaked out of the block
console.log(b); // ReferenceError — b never existed outside the block
```

**`const` locks the binding, not the value:**
```js
const arr = [1, 2, 3];
arr.push(4);   // fine — mutating the array is allowed
arr = [5, 6];  // TypeError — reassigning the binding is not
```
`const` means "can't be reassigned," not "immutable" — a common misconception.

## Hoisting

```js
console.log(x); // undefined, not an error
var x = 5;

console.log(y); // ReferenceError
let y = 5;
```
All three are hoisted, but differently: `var` is hoisted to the top of its function scope and initialized to `undefined` immediately — referencing it early just gives `undefined`. `let`/`const` are hoisted (name registered in scope from block start) but not initialized until the actual declaration line executes.

## Temporal dead zone (TDZ)

```js
{
  console.log(typeof z); // ReferenceError, not "undefined"
  let z = 1;
}
```
The gap between block start and the actual `let`/`const` declaration — accessing the variable in that window throws `ReferenceError` rather than returning `undefined`. A deliberate design decision: `var`'s "hoisted but undefined" behavior caused subtle bugs (using a variable before intended initialization silently gave `undefined` instead of failing loudly); `let`/`const` fail fast instead.

## What a closure actually is

A function that "remembers" variables from the scope it was created in, even after that outer scope finishes executing.

```js
function makeCounter() {
  let count = 0;
  return function() {
    count++;
    return count;
  };
}
const counter = makeCounter();
counter(); // 1
counter(); // 2
```
The returned inner function keeps a live reference to `count` — not a copy of its value, an ongoing reference to that specific variable. The closure "closes over" the variable environment it was created in.

**Core mental model: closures capture a reference to a variable, not a value.** The question is always "how many separate variables exist," not "how do closures behave differently."

## The classic loop closure bug — worked through in detail

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// logs: 3, 3, 3

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// logs: 0, 1, 2
```

**With `var`:** function-scoped, and a `for` loop doesn't create a new function scope per iteration — there's exactly ONE `i` shared across the whole loop, mutated in place each pass. All three arrow functions close over that same single variable. By the time the 100ms timeouts fire, the loop has finished and `i` sits at `3` — all three closures read `3`.

**With `let`:** the spec defines that each loop iteration gets its own fresh binding of `i`, copied forward from the previous iteration's value at the start of each pass — three distinct variables in three distinct scopes. Each arrow function closes over a *different* `i`, so callbacks read `0`, `1`, `2` respectively.

One of the most commonly asked "explain the difference" interview questions, since it demonstrates hoisting, scoping, and closures together in one example.

## Shared vs separate closure instances — worked example (event listeners)

**Case 1 — one `makeCounter()` call, shared state, count reaches 2:**
```js
const counter = makeCounter(); // ONE call, ONE count variable
button1.addEventListener('click', counter);
button2.addEventListener('click', counter);
// click button1 → count becomes 1
// click button2 → SAME count becomes 2
```

**Case 2 — two separate `makeCounter()` calls, independent state, each stays at 1:**
```js
const counter1 = makeCounter(); // own count
const counter2 = makeCounter(); // different count
button1.addEventListener('click', counter1);
button2.addEventListener('click', counter2);
// click button1 → its own count becomes 1
// click button2 → a completely different count becomes 1, not 2
```
**The test to apply:** did the counter-creating function get called once and reused for both listeners (one shared variable), or called separately per listener (independent variables)?

## "Same instance, but reset per use" — a genuinely different, valid pattern

A single shared closure can't automatically give each listener an independent starting point — that contradiction is resolved by recognizing that "independent starting point per listener" is just what separate instances (Case 2) already are. But a single shared instance CAN deliberately expose a reset capability if that's the actual need (one shared counter, intentionally reset before certain uses):

```js
function makeCounter() {
  let count = 0;
  return {
    increment() { count++; return count; },
    reset() { count = 0; }
  };
}
const counter = makeCounter();
button1.addEventListener('click', () => counter.increment());
button2.addEventListener('click', () => { counter.reset(); counter.increment(); });
```
Still one shared `count`, but explicitly reset before button2's increment, every click. JavaScript gives no automatic per-listener isolation on a shared instance, but nothing prevents manually building reset logic, since a closure's variable is just a normal variable any function with access to it can reassign.

**Design question to ask:** do the listeners represent genuinely separate counters (→ separate instances, Case 2) or one shared counter needing intentional reset behavior (→ shared instance + reset method)? Different designs for different problems, not two ways of writing the same thing.

## Practical rule

Modern JS style prefers `const` by default, `let` only when reassignment is genuinely needed, `var` recognized in legacy code but never written. Block scoping and the TDZ genuinely prevent real bug classes that `var`'s looser behavior allowed.

## Where we left off

First slice of JS Section 1 covered in depth: var/let/const, hoisting, TDZ, and a thorough closures deep dive (the loop bug, shared vs separate closure instances via event listener examples, and the reset pattern for shared state). Remaining Section 1 slices: data types (primitives vs objects), type coercion + `==` vs `===`, operators (including `??`/`?.`), truthy/falsy values, template literals/tagged templates.
