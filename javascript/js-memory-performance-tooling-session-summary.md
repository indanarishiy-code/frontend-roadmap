# Session Summary: Memory & Performance + Leak Detection Tooling (JS Section 13)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Garbage collection — mark-and-sweep

```js
let obj = { data: "large" };
obj = null; // original object now unreachable, eligible for GC
```
JS uses mark-and-sweep: starting from "roots" (global variables, currently executing scopes), the engine marks every reachable object; anything unmarked gets swept and reclaimed. **Key insight:** memory isn't freed when a variable goes out of scope per se — it's freed when nothing anywhere still holds a reference. This is why JS memory leaks are almost always "something is still holding a reference I forgot about," not manual memory-management error.

## Memory leaks — common real causes

**1. Detached DOM references:**
```js
let cachedElement = document.getElementById('widget');
someContainer.innerHTML = ''; // widget visually gone
// cachedElement still references it — can't be garbage collected
```

**2. Forgotten timers/listeners — directly relevant to Vue work:**
```js
onMounted(() => {
  const interval = setInterval(() => { updateData(); }, 1000);
  window.addEventListener('resize', handleResize);
});
onUnmounted(() => {
  clearInterval(interval);
  window.removeEventListener('resize', handleResize);
});
```
A component setting up an interval/listener in `onMounted` without teardown in `onUnmounted` leaks memory on every create/destroy cycle — the interval callback closure keeps the entire component instance's scope alive indefinitely.

**3. Closures holding large scope:**
```js
function setup() {
  const hugeData = new Array(1000000).fill("data");
  const smallFn = () => console.log("hi"); // doesn't use hugeData
  return smallFn;
}
const fn = setup(); // hugeData STILL kept alive — same closure scope as smallFn
```
A closure keeps its *entire* enclosing scope alive, not just the variables it actually references.

## `structuredClone()` — native deep cloning

```js
const clone = structuredClone(original);
```
Directly solves the shallow-copy problem from spread/`Object.assign()` (Section 3) — a genuine, native, full deep clone. Replaces the old `JSON.parse(JSON.stringify(obj))` hack, which silently drops functions, breaks on `Date` objects, and can't handle circular references. `structuredClone()` correctly handles `Date`, `Map`, `Set`, and circular references (still can't clone functions or DOM nodes).

## Debouncing and throttling

```js
function debounce(fn, delay) {
  let timeoutId;
  return (...args) => { clearTimeout(timeoutId); timeoutId = setTimeout(() => fn(...args), delay); };
}
```
**Debounce** — delays execution until a pause in activity. Essential for search-as-you-type, form validation on input.

```js
function throttle(fn, limit) {
  let inThrottle;
  return (...args) => { if (!inThrottle) { fn(...args); inThrottle = true; setTimeout(() => inThrottle = false, limit); } };
}
```
**Throttle** — guarantees execution at most once per time window regardless of event frequency. Right tool for scroll/resize/mousemove.

**The distinction:** debounce = "wait until done," throttle = "limit frequency, keep updating during continuous activity" — genuinely different use cases; picking the wrong one produces a noticeably wrong UX.

## Memoization

```js
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```
Caches a pure function's results by argument. **This is the same underlying concept behind Vue's `computed` properties** — caches its result, only recalculates when reactive dependencies change — memoization applied automatically by the framework.

## Recommended libraries/tools for these utilities (verified via search)

**VueUse (`@vueuse/core`)** — the idiomatic default for Vue 3 work:
```js
import { useDebounceFn, useThrottleFn, useEventListener } from '@vueuse/core'
const debouncedSearch = useDebounceFn((query) => searchAPI(query), 300);
useEventListener(window, 'resize', handleResize); // auto-cleanup on unmount, no onUnmounted needed
```
Comprehensive Vue composables, SSR-friendly, TypeScript support, tree-shakable. `useEventListener`/`useIntervalFn` wire teardown in automatically via `tryOnScopeDispose`, directly preventing the forgotten-listener leak pattern.

**Real gotcha:** debounced handlers must be created in `setup()` or as module-level constants — never as inline arrow functions in templates. Vue re-evaluates template expressions on reactive changes, so inline `debounce(fn, 300)` in a template creates a fresh debounced function on every re-render, silently breaking the debounce.

**Lodash** — still valid, especially for existing codebases already depending on it, or for plain (non-reactive) memoization outside Vue's reactivity system.

## Detecting leaks AFTER the fact — runtime profiling tools (verified via search)

**Chrome DevTools Memory panel** — the primary, built-in tool:
- **Heap snapshots** — take before/after an action, use Compare view; filter Class for "Detached" to find detached DOM trees specifically
- **Allocation Timeline** — records every allocation with a stack trace, showing exactly which function created a leaking object
- **Warning sign:** a sawtooth memory graph that never returns to baseline across repeated actions (open/close, navigate back and forth) is the visual signature of a real leak

**Chrome DevTools MCP** — genuinely new, relevant if using an agentic coding tool (Claude Code, etc.): automates heap snapshot collection/comparison with purpose-built categories — `objectsRetainedByDetachedDomNodes`, `objectsRetainedByEventHandlers`, `objectsRetainedByContexts` — an agent can navigate a scenario, take snapshots, and directly report something like "a scroll listener on window was never cleared."

**Edge DevTools** — has a dedicated Detached Elements tool, a more targeted version of the same detection.

## Detecting leaks BEFORE runtime — static analysis / linting (verified via search, genuinely important gap identified)

**The honest current state:** a 2026 empirical study across 500 public repos found real, quantified coverage gaps. **Vue specifically has no official ESLint rule for unstored watch stop handles** — Vue DevTools can show active watchers but won't flag missing cleanup in code review. `setTimeout` without `clearTimeout` accounted for 40% of all findings across the study (22,384 instances) — this tooling gap is why these patterns persist at scale despite being well-understood.

**What DOES exist and is worth installing:**
- **`@antfu/eslint-config`** (via `eslint-plugin-antfu`) has genuinely relevant rules: enforces `addEventListener` has a paired `removeEventListener`, `setInterval` has a paired `clearInterval`, `IntersectionObserver`/`ResizeObserver` have paired `.disconnect()`, `fetch` has a paired `AbortController` abort in cleanup
- **`eslint-plugin-vue`'s `vue/no-watch-after-await`** — ships in the essential preset, but only catches one narrow case (watcher registered after an `await`), not general missing-cleanup patterns
- React's ecosystem has somewhat more coverage (`eslint-plugin-react-hooks`, newer tools like `loctree`'s async-effect-cleanup detector), but that's React-only, not applicable to Vue

**The practical, honest recommendation given this real gap:** since comprehensive static coverage genuinely doesn't exist for Vue yet, the most reliable current mitigation is architectural rather than purely tooling-based — default to VueUse's composables (`useEventListener`, `useIntervalFn`) instead of raw `addEventListener`/`setInterval`, since they wire teardown in automatically via `tryOnScopeDispose`. This means there's no manual cleanup step to forget in the first place, rather than relying on a linter that doesn't comprehensively catch this yet. Installing `@antfu/eslint-config`'s relevant rules provides partial coverage worth having on top of the VueUse-by-default habit.

## Where we left off

Section 13 (Memory & Performance) covered in full, extended by a genuinely thorough, honestly-caveated investigation into actual current tooling: VueUse/Lodash for the utility patterns themselves, Chrome DevTools Memory panel and Chrome DevTools MCP for after-the-fact runtime detection, and a real, verified assessment of static-analysis/linting coverage gaps specifically for Vue (no official rule for unstored watch handles), concluding that VueUse-by-default is currently the more reliable practical defense than waiting for comprehensive linting coverage.

Next up per the JavaScript reference doc: **Section 14 — The DOM & Browser APIs** (DOM traversal/manipulation, event handling/bubbling/capturing/delegation, custom events, `fetch()`/Headers/Request/Response, localStorage/sessionStorage/cookies, history API, IntersectionObserver/ResizeObserver/MutationObserver, requestAnimationFrame, Web Workers, Intl API).
