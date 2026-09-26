# Session Summary: Observers, requestAnimationFrame, Web Workers, Intl API (JS Section 14, Part 3)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `IntersectionObserver`

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      loadImage(entry.target);
      observer.unobserve(entry.target);
    }
  });
}, { threshold: 0.1 });
```
**Problem solved:** before this, detecting "has this element scrolled into view" required manually listening to `scroll` events and calculating positions with `getBoundingClientRect()` on every scroll event — expensive, since scroll fires extremely frequently (why throttle was so commonly paired with scroll handlers). `IntersectionObserver` runs asynchronously, off the main thread's critical path, firing only when visibility actually changes — no manual position math or throttling needed.

**Practical relevance:** lazy-loading images/data on scroll, infinite-scroll pagination, triggering an animation only once an element is visible.

## `ResizeObserver`

```js
const observer = new ResizeObserver((entries) => {
  entries.forEach(entry => { entry.contentRect.width; entry.contentRect.height; });
});
observer.observe(document.querySelector('.chart-container'));
```
**Genuinely different from `window.addEventListener('resize', ...)`:** the native `resize` event only fires on viewport changes — zero awareness of an individual element resizing from layout changes, sidebar toggling, flex/grid reflow, or content changes affecting just that element. `ResizeObserver` watches a specific element directly — relevant for responsive charts/graphs that need to redraw whenever their container size changes, regardless of why.

## `MutationObserver`

```js
const observer = new MutationObserver((mutations) => {
  mutations.forEach(m => { m.type; m.addedNodes; m.removedNodes; });
});
observer.observe(target, { childList: true, subtree: true });
```
**Genuinely niche in typical Vue work:** in a framework-managed app, rarely needed since Vue itself already knows when it changes the DOM. More relevant for vanilla JS/library code reacting to DOM changes made by code you don't control (third-party widgets, framework-agnostic libraries). Worth recognizing the API exists, not expecting frequent use.

## `requestAnimationFrame`

```js
function animate() {
  element.style.transform = `translateX(${position}px)`;
  position += 2;
  if (position < 500) requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```
**Why it beats `setTimeout`/`setInterval` for animation:** synchronized with the browser's actual repaint cycle (typically 60fps, matching display refresh) — the browser calls the function right before painting the next frame, so visual updates are genuinely smooth and never wasted (no updates while the tab is backgrounded, unlike `setInterval`). The JS-side equivalent of the `transform`/`opacity` compositor-performance principle from the CSS animations session — work *with* the browser's rendering cycle, not against it with an arbitrary timer.

## Web Workers — genuine off-main-thread execution

```js
// main.js
const worker = new Worker('worker.js');
worker.postMessage({ numbers: [...] });
worker.onmessage = (event) => { event.data; };
// worker.js
self.onmessage = (event) => {
  const result = event.data.numbers.reduce((sum, n) => sum + n, 0);
  self.postMessage(result);
};
```
**Problem solved:** JS is single-threaded on the main thread — heavy computation blocks the entire UI (freezes clicks/scroll/render) until it finishes. Web Workers run JS on a genuinely separate thread, communicating only via message-passing (`postMessage`/`onmessage`) — can't touch the DOM directly, but can compute heavily without freezing the UI.

### When frontend actually needs this (clarified — most dashboard work does NOT)

**Honest starting point:** most CRUD/dashboard work (fetch from API → display → edit → submit) correctly relies on the backend for computation — Web Workers are a narrower-need tool than "any data calculation" might suggest.

**Concrete real categories where frontend-side heavy computation genuinely happens:**
1. **Client-side filtering/sorting/aggregation on an already-loaded large dataset** — deliberately loading a large chunk upfront so subsequent interactions (filter, sort, group) are instant without a network round-trip per change. Relevant if a dashboard loads a large historical dataset once and lets users slice/filter it locally without re-querying the backend each time.
2. **Image/file processing entirely in-browser** — resizing/compressing images before upload, parsing/validating a large CSV/Excel upload before deciding whether to send it, generating a PDF/export client-side.
3. **Real-time visualization with heavy rendering math** — recalculating statistical transformations (moving averages, regressions) across thousands of points as a user drags a time-range slider, where recalculating per drag-frame would freeze the UI.
4. **Search/autocomplete over a large in-memory dataset** — fuzzy-matching over tens of thousands of pre-loaded options for instant results without a per-keystroke server request.
5. **Client-side cryptography/hashing** — hashing a password before sending, verifying a signature in-browser.

**The common pattern across all of these:** the data is already in the browser AND the computation needs to happen without a server round-trip — either for deliberately instant/offline-feeling interactions, or because the data genuinely shouldn't go back to the server yet. A normal "request → backend computes → response → display" workflow correctly doesn't need this.

**Why worth knowing anyway:** there's a real industry trend toward more client-side computation for responsiveness in data-heavy dashboard tools — if oil-and-gas dashboards ever load a large dataset once (e.g. a month of sensor readings) and let users interactively slice/chart it without re-fetching per interaction, that's precisely the scenario to reach for this. A "know it exists, know exactly when the need arises" tool rather than something to proactively seek uses for.

## Intl API — native internationalization, directly relevant to target markets

```js
new Intl.NumberFormat('de-DE', { style: 'currency', currency: 'EUR' }).format(1234.5);
// "1.234,50 €" — German: period for thousands, comma for decimal
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(1234.5);
// "$1,234.50" — US: comma for thousands, period for decimal
new Intl.DateTimeFormat('de-DE').format(new Date());
new Intl.Collator('de-DE').compare('ä', 'z'); // locale-aware string sorting/comparison
```
**Why directly relevant:** given Germany/Netherlands/Malaysia/UAE as target markets, number/currency/date formatting conventions genuinely differ significantly between locales — `Intl` is the native, no-library way to handle this correctly rather than manually string-formatting (error-prone, easy to get subtly wrong for unfamiliar locales). Connects directly to Temporal (Section 9) — `Temporal` objects can be formatted with `Intl.DateTimeFormat` for locale-correct display, completing the date/time/formatting story together.

## Where we left off

Third slice of Section 14 covered: all three Observer APIs, `requestAnimationFrame`'s compositor-sync rationale, a thorough and honestly-scoped discussion of when frontend work actually needs Web Workers (narrower than "any computation" — specifically already-loaded-client-side-data scenarios), and the Intl API's direct relevance to multi-locale target markets.

**Remaining Section 14:** this closes out the section's core content — the observer APIs, requestAnimationFrame, Web Workers, and Intl API were the last pieces. Section 14 (The DOM & Browser APIs) is now fully covered across three sessions (DOM traversal/events, fetch/storage/history, observers/workers/i18n).

Next up per the JavaScript reference doc: **Section 15 — Sets, Maps & Collections** (`Set`/`WeakSet`, `Map`/`WeakMap`, new ES2025 Set methods — `union`/`intersection`/`difference`/etc. — `Map.prototype.getOrInsert()`, when to use `Map`/`Set` over plain objects/arrays).
