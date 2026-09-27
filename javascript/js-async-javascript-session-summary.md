# Session Summary: Asynchronous JavaScript (Section 10)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## The event loop

JS is single-threaded — one call stack. Async doesn't mean multi-threaded execution; it means the runtime (browser/Node) hands off work and reschedules the callback later.

**The pieces:**
- **Call stack** — synchronous code executes here, frame by frame
- **Web APIs / Node APIs** — where the actual async work happens outside the JS thread (timers, network, I/O)
- **Macrotask queue** — `setTimeout`, `setInterval`, I/O callbacks, UI events
- **Microtask queue** — Promise `.then`/`.catch`/`.finally` callbacks, `queueMicrotask`, `async`/`await` continuations

**The loop's actual order, per tick:**
1. Run everything currently on the call stack until it's empty
2. Drain the **entire** microtask queue — not just one task: if a microtask queues another microtask, that new one also runs before moving on. The queue must be truly empty before proceeding.
3. Only then take **one** task off the macrotask queue, run it, then go back to step 2

This is why this always logs `A, D, C, B` (not `A, B, C, D`):

```js
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
```

`setTimeout(..., 0)` does **not** mean "run immediately" — it means "run after the stack clears and all microtasks drain," which is why microtasks (Promises) always beat a `0ms` timeout.

## Callback hell → why Promises exist

Nesting callbacks for sequential async steps produces the classic pyramid:

```js
getUser(id, (user) => {
  getPosts(user.id, (posts) => {
    getComments(posts[0].id, (comments) => {
      // 3 levels deep, error handling duplicated at each level
    });
  });
});
```

Problems: unreadable nesting, error handling repeated at every level (no single `catch`), inversion of control (you hand your callback to code you don't control and trust it to call it correctly, once, with the right arguments).

## Promise state machine

A Promise has exactly one of three states, and a transition is **one-way and permanent**:
- `pending` → `fulfilled` (with a value) — can never happen again or change
- `pending` → `rejected` (with a reason) — same

Once settled (fulfilled or rejected), calling `resolve`/`reject` again does nothing — state is locked.

```js
const p = new Promise((resolve, reject) => {
  doAsyncThing((err, result) => {
    if (err) reject(err);
    else resolve(result);
  });
});
```

**Chaining (`.then`):** each `.then` returns a **new** Promise. If the callback returns a plain value, the new Promise resolves with it. If it returns another Promise, the chain "flattens" — it waits for that inner Promise before resolving. This is why chains stay flat instead of nesting:

```js
fetch(url)
  .then(res => res.json())      // returns a Promise → chain waits for it
  .then(data => processData(data))
  .catch(err => handleError(err)); // catches a rejection from ANY prior step
```

## The four Promise combinators — with real use cases

| Combinator | Settles when | Use case |
|---|---|---|
| `Promise.all()` | all fulfill, or **first** rejection (short-circuits) | Need every result; any failure invalidates the whole batch (e.g. loading several required config files) |
| `Promise.allSettled()` | always — once all have settled, fulfilled or rejected | Want every outcome regardless of individual failures (e.g. submitting to 5 analytics endpoints — one failing shouldn't hide the other 4 results) |
| `Promise.race()` | **first** settled promise, whether fulfilled or rejected | **Manual timeout pattern** — race a real request against a timer that rejects: `Promise.race([fetch(url), timeoutPromise])` — whichever finishes (or times out) first wins |
| `Promise.any()` | **first fulfillment**; only rejects if **all** reject (`AggregateError`) | Redundant/fallback data sources — e.g. querying 3 mirror APIs or CDN endpoints for the same resource; take whichever succeeds first, ignore individual failures |

The `race` vs `any` distinction is the practical one: `race` cares about *first to settle at all* (good for timeouts, where a rejection should also "win"); `any` cares about *first success* (good for redundancy, where you only care about failure if everything failed).

## `Promise.withResolvers()` and `Promise.try()`

Newer additions that solve real ergonomic pain points:

```js
// Before: had to capture resolve/reject via closure from inside the executor
let resolve, reject;
const p = new Promise((res, rej) => { resolve = res; reject = rej; });

// Now: resolve/reject exposed directly, no executor needed
const { promise, resolve, reject } = Promise.withResolvers();
```
Useful when you need to resolve a Promise from an event handler or external callback that isn't naturally inside the executor's scope.

```js
// Promise.try() — wraps a function that might throw synchronously OR return a Promise,
// normalizing both into a single Promise chain
Promise.try(() => mightThrowOrReturnPromise())
  .then(...)
  .catch(...); // catches BOTH sync throws and async rejections uniformly
```

## `async`/`await`

Syntactic sugar over Promises — not a different mechanism.

**Structural fact:** an `async` function **always** returns a Promise, even if you `return` a plain value or nothing at all:
```js
async function f() { return 42; }
f(); // Promise, which resolves to 42 — NOT 42 directly
```

`await` pauses the async function's execution (not the whole thread) until the awaited Promise settles, then either returns the value or throws the rejection reason — which is why plain `try`/`catch` works around `await`.

### Sequential vs parallel awaiting — the real-world performance mistake

```js
// SEQUENTIAL — each await blocks the next; total time = sum of all three
const user = await getUser();
const posts = await getPosts();
const comments = await getComments();

// PARALLEL — all three start immediately; total time = the SLOWEST one
const [user, posts, comments] = await Promise.all([
  getUser(), getPosts(), getComments()
]);
```
Sequential is only correct when a later call genuinely **depends on** an earlier result. Awaiting independent calls one at a time is a common, easy-to-miss performance bug.

### Top-level await

Allowed directly at a module's top level (ES modules only, not CommonJS/scripts) — no wrapping `async` IIFE needed:
```js
// inside a .mjs or type="module" file
const config = await fetchConfig();
```
A module using top-level await blocks any module that imports it until it resolves — worth knowing as a loading-order implication, not just a syntax convenience.

## Unhandled Promise rejections — best practices (verified via search)

**The core question:** can a rejection "fall through the cracks" if nothing `.catch()`es it? Yes — and browsers/Node fire a dedicated event for exactly this.

**Global handling in plain JS/browser:**
```js
window.addEventListener("unhandledrejection", (event) => {
  console.error("Unhandled rejection:", event.reason);
  // send to error-monitoring service
});
```

**The Vue-specific finding:** Vue's `app.config.errorHandler` is **not** a complete safety net for async errors. It reliably catches synchronous errors thrown inside lifecycle hooks, render functions, and watchers — but there's a documented gap: Promise rejections surfacing from certain async contexts (notably async `watch`/`watchEffect` callbacks) can bypass it entirely, and it has **zero visibility into anything outside Vue's own execution context** (a rejected fetch in an unrelated utility function, a timer callback, etc.).

**Correct combined setup** for real error monitoring coverage:
```js
app.config.errorHandler = (err, instance, info) => {
  // catches most sync errors inside Vue's own lifecycle
  reportError(err, { source: "vue", info });
};

window.addEventListener("unhandledrejection", (event) => {
  // catches Promise rejections Vue's handler misses, and anything outside Vue entirely
  reportError(event.reason, { source: "unhandledrejection" });
});

window.addEventListener("error", (event) => {
  // catches uncaught sync errors outside Vue's context (e.g. plain <script> code)
  reportError(event.error, { source: "window" });
});
```
All three wired to the same error-monitoring service (Sentry, etc.) is the practical recommendation — none of the three alone is sufficient coverage.

## Where we left off

Section 10 (Asynchronous JavaScript) closed at senior-frontend depth: event loop ordering, Promise mechanics and all four combinators (including real `race`/`any` use cases), the newer `withResolvers`/`try` helpers, async/await semantics and the sequential-vs-parallel performance distinction, top-level await, and a verified answer on unhandled-rejection handling including the specific Vue `errorHandler` gap. Next up: **Section 18 — Asynchronous & Data-Fetching Patterns in Practice** (loading/error/success state handling, race-condition/stale-request handling, `AbortController`/`AbortSignal` for cancelable fetches).
