# Session Summary: Functional Programming Patterns (JS Section 17)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Immutability principles — the unifying philosophy across the whole JS study

The functional principle underneath everything already covered — spread syntax (Section 2), `toSorted()`/`toReversed()`/`with()` (Section 5), `structuredClone()` (Section 13): **never mutate data in place; always produce a new value representing the change.**

```js
// Mutating (imperative)
function addItem(cart, item) {
  cart.items.push(item); // mutates the original
  return cart;
}
// Immutable (functional)
function addItem(cart, item) {
  return { ...cart, items: [...cart.items, item] }; // returns a genuinely NEW cart
}
```
**Practical importance beyond principle:** connects directly to Pinia's reactivity and Vue's change detection — predictable state updates, reliable undo/redo, and easier debugging (any logged state snapshot is guaranteed never silently mutated afterward) all depend on this discipline. Every "recent gap" immutability-related feature flagged throughout this JS study exists specifically in service of this one underlying principle.

## Function composition

```js
const double = (x) => x * 2;
const addOne = (x) => x + 1;
const square = (x) => x * x;

// Manual composition — awkward past 2-3 functions
const result = square(addOne(double(5))); // 121

// compose() helper — combines right-to-left
function compose(...fns) {
  return (x) => fns.reduceRight((acc, fn) => fn(acc), x);
}
const transform = compose(square, addOne, double);
transform(5); // 121
```
**Genuine value:** builds complex transformations from small, individually-testable, individually-reusable pieces rather than one large function doing everything — the same pure-function philosophy from Section 2, applied to composing larger behavior from small pure pieces.

## Currying vs partial application — genuinely different, often conflated

**Currying** — transforms a multi-argument function into a chain of exactly one-argument functions:
```js
function curriedAdd(a) { return (b) => (c) => a + b + c; }
curriedAdd(1)(2)(3); // 6
```

**Partial application** — fixes SOME arguments upfront (any number), returns a function taking the rest in one call, not necessarily one at a time:
```js
function partial(fn, ...presetArgs) {
  return (...laterArgs) => fn(...presetArgs, ...laterArgs);
}
const addFive = partial(add, 5); // fixes 'a' as 5
addFive(2, 3); // 10 — takes the REMAINING two at once
```

**The genuine distinction:** currying always produces a chain of strictly one-argument functions; partial application fixes any number of arguments and returns a function for the rest, in one call. Currying is a specific, strict form of the more general partial-application idea.

**Practical real use case — currying slots directly into higher-order functions:**
```js
const multiply = (a) => (b) => a * b;
const double = multiply(2);
[1, 2, 3].map(double); // [2, 4, 6] — pre-configured with one argument, ready to hand to map()
```

## The pipe operator — proposal status, and today's manual pattern

```js
// PROPOSED syntax (NOT yet standard JS — Stage 2 proposal)
const result = 5 |> double |> addOne |> square;

// The manual pattern used TODAY, since the operator isn't available yet
function pipe(...fns) {
  return (x) => fns.reduce((acc, fn) => fn(acc), x); // reduce, not reduceRight — LEFT-to-right
}
const transform = pipe(double, addOne, square);
transform(5); // 121 — same result as compose(), reads in the order you'd say it aloud
```
**Genuinely important distinction between `pipe` and `compose`:** `compose` combines right-to-left (matches nested function calls, `f(g(x))`); `pipe` combines left-to-right (matches narrating the steps out loud: "first double, then add one, then square"). This readability difference is exactly why the pipe operator proposal exists — to make this common pattern a native feature instead of a hand-rolled utility every project reimplements slightly differently.

**Current practical status:** since `|>` isn't standard yet, the `pipe()` helper function (or a small library utility) is genuinely how this is done in real projects today — recognize the proposed syntax if seen, but the manual `pipe()` function is the actual current tool.

## Where we left off

Section 17 (Functional Programming Patterns) covered in full — immutability tied together as the unifying principle behind several previously-covered ES features, function composition, the precise currying-vs-partial-application distinction (a genuinely easy pair to conflate), and the pipe/compose direction difference plus the pipe operator's current proposal (not-yet-standard) status.

Next up per the JavaScript reference doc: **Section 18 — Asynchronous & Data-Fetching Patterns in Practice** (handling loading/error/success states manually, race condition handling and stale request cancellation, `AbortController`/`AbortSignal` for cancelable fetches) — genuinely practical given actual dashboard/data-fetching work.
