# Session Summary: Modules (JS Section 10)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## ES Modules — named, default, namespace imports

```js
// utils.js
export const add = (a, b) => a + b;
export default function multiply(a, b) { return a * b; }

// consumer.js
import multiply, { add } from './utils.js'; // default + named together
import * as utils from './utils.js';           // namespace import — everything as one object
```

**Named vs default — genuinely important distinction:**
```js
export const add = (a, b) => a + b;
import { add } from './utils.js';          // MUST match exact name (or alias)
import { add as sum } from './utils.js';    // aliasing

export default function() {}
import anythingYouWant from './utils.js';    // default export — importer names it freely, no matching required
```
**Common confusion:** default exports don't enforce a name match, which is flexible but means no automatic "does this even exist" check the way named exports get — a typo in a default import can silently import `undefined` in some tooling configurations, rather than throwing.

## Dynamic `import()`

```js
button.addEventListener('click', async () => {
  const { default: Modal } = await import('./Modal.js'); // loads ONLY when needed
  new Modal().open();
});
```
Unlike static `import` (top of file, resolved at build/parse time), `import()` is a genuine function call returning a Promise, callable conditionally or inside event handlers. **This is the actual mechanism behind code-splitting** — Vite/webpack split a dynamically-imported module into its own bundle chunk, downloaded only when the `import()` call executes.

**Directly relevant to Vue Router lazy-loaded routes — already used, possibly without naming it:**
```js
const routes = [
  { path: '/dashboard', component: () => import('./views/Dashboard.vue') } // only loaded when route is visited
];
```

## `import defer` — genuinely new, niche

```js
import defer * as heavyModule from './heavy-module.js';
```
**Problem solved:** a normal static `import` executes the imported module's top-level code immediately on load, even if unused in a given code path. `import defer` delays actual *evaluation* until first access, while still resolving the module reference eagerly — a middle ground between static import (always immediate) and dynamic `import()` (fully deferred, requires async). Very recent, niche — recognize the name/problem category, not yet broadly adopted.

## CommonJS — why it exists, why ESM won

```js
// CommonJS (Node.js legacy)
const { add } = require('./utils.js');
module.exports = { multiply };

// ES Modules (modern standard)
import { add } from './utils.js';
export const multiply = ...;
```
CommonJS was Node's original module system, predating ESM being part of the language itself (ESM standardized ES2015, took years for full Node support).

**Why ESM genuinely won, not just convention:** CommonJS's `require()` is synchronous, resolved at runtime — makes static analysis (and tree-shaking) essentially impossible, since a bundler can't know what `require()` resolves to without running the code (it can be conditional, dynamically-constructed path, inside any function). ESM's `import`/`export` are required at the top level (not inside conditionals/functions) specifically so bundlers can statically analyze the whole dependency graph before running anything.

**Practical relevance:** still encountered in older Node backend code, unmigrated npm packages, some Node config files — worth recognizing the syntax and the *why* behind the difference, for any backend/Node tooling work.

## Module scope

```js
// file1.js
const secret = "hidden";
// file2.js
console.log(secret); // ReferenceError — no access to file1's top-level variables at all
```
Every ES module has its own private scope automatically — the same concept IIFEs (Section 2) existed to fake before native modules did it. Nothing leaks out of a module's top-level scope unless explicitly `export`ed.

## Tree-shaking implications — the practical payoff

```js
// utils.js — exports many functions
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
// ... 50 more

// consumer.js
import { add } from './utils.js'; // only importing ONE function
```
**What tree-shaking does:** because ESM's static, top-level structure lets a bundler know exactly what's imported/exported without running code, it can determine `subtract` and the other 50 functions are never used and **exclude them entirely from the production bundle** — a genuine, measurable size reduction, not theoretical. This is exactly why libraries structured with many small granular exports (ESM-first) tend to produce smaller final bundles than CommonJS equivalents, and why ESM's "no conditional imports, no dynamic require paths" static-structure requirement exists as a deliberate language design constraint, not an arbitrary restriction.

## Where we left off

Section 10 (Modules) covered in full — the named/default distinction, dynamic `import()` (directly tied to Vue Router lazy-loading already used in practice), `import defer` as a recent niche addition, the real reason ESM's static structure beat CommonJS (enabling tree-shaking, not just being newer), module scope, and the practical tree-shaking payoff.

Next up per the JavaScript reference doc: **Section 11 — Error Handling** (try/catch/finally, custom error classes extending Error, error types, error handling in async code specifically, `using` declarations/Explicit Resource Management — a newer feature for deterministic cleanup).
