# Session Summary: fetch/Storage/History APIs + fetch vs Axios (JS Section 14, Part 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `fetch()` — the critical gotcha: does NOT reject on HTTP error statuses

```js
const response = await fetch('/api/users/999'); // server returns 404
response.ok;      // false
response.status;   // 404
// fetch() did NOT throw or reject — no error raised

const data = await response.json(); // runs FINE, even for a 404 response
```
`fetch()` only rejects for genuine network-level failures (DNS failure, no connection, CORS block). A 404/500/any HTTP error status is still a "successful" fetch as far as the Promise is concerned — the server responded, just with an error status.

**Correct standard pattern — always check `response.ok` explicitly:**
```js
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error(`HTTP ${response.status}: ${response.statusText}`);
  return response.json();
}
```
Skipping this is a genuinely common real bug — code assuming `fetch()` throwing means "something went wrong" silently treats a 404/500 as success, then crashes later using error-shaped JSON as valid data.

## Headers, Request, Response objects

```js
const headers = new Headers();
headers.append('Authorization', `Bearer ${token}`);
const request = new Request('/api/users', { method: 'POST', headers, body: JSON.stringify({ name: "Alex" }) });
const response = await fetch(request); // can pass a Request object directly instead of a URL string
```
Exist as separate constructable objects so a request config can be built/reused independently of sending it — useful for retry logic or a reusable base template in an API client wrapper.

```js
response.headers.get('Content-Type');
response.clone(); // Response bodies can only be read ONCE — clone() gives a second independent copy to read separately
```
**Real gotcha:** calling `.json()` consumes the body stream — calling it again throws. `response.clone()` is needed if a body must be inspected two different ways (raw text logging AND JSON parsing, for instance).

**Practical relevance:** likely abstracted away by Axios/Pinia store actions in actual work, but understanding these primitives helps when debugging network issues, CORS errors, or writing a custom fetch wrapper/interceptor.

## `localStorage`/`sessionStorage`

```js
localStorage.setItem('theme', 'dark');
sessionStorage.setItem('draft', JSON.stringify({ text: "unsaved" })); // must manually stringify/parse objects
```
- **`localStorage`** — persists indefinitely across browser restarts until explicitly cleared
- **`sessionStorage`** — persists only for the current tab's session, per-tab (not shared across tabs of the same site)
- **Both only store strings** — objects/arrays must be manually `JSON.stringify()`d before storing, `JSON.parse()`d after retrieving; storing an object directly silently stores `"[object Object]"`
- **Both synchronous, blocking, with a real size limit** (~5-10MB per origin) — not for large datasets, only small config/preference/draft data

## Cookies via `document.cookie` — genuinely awkward, why it's rarely hand-rolled

```js
document.cookie = "theme=dark; expires=Fri, 31 Dec 2026 23:59:59 GMT; path=/";
document.cookie; // ALL cookies as one concatenated string, semicolon-separated
```
Reading returns every cookie as one string requiring manual parsing; writing merges in just the one cookie specified rather than replacing all (an unusual "magic setter" unlike any other DOM API). This is why most real projects use a small cookie library rather than hand-rolling parsing — and part of why cookies are increasingly reserved for server-visible needs (auth tokens sent automatically with requests), with `localStorage` handling client-only data instead.

## History API — the foundation under Vue Router

```js
history.pushState({ page: 1 }, '', '/users/42'); // changes URL WITHOUT a full page reload
window.addEventListener('popstate', (event) => { event.state; }); // fires on back/forward navigation
```
This is genuinely the mechanism Vue Router is built on when using default "history mode" (vs hash-based routing). `pushState()` changes the URL and adds a history entry without a real navigation/reload — what makes SPAs have real, bookmarkable, back-button-compatible URLs while staying a single loaded JS app. `popstate` fires on back/forward navigation, letting the router respond by rendering the correct view without a reload.

**Practical server-config implication:** a Vue app using history-mode routing requires the server to return `index.html` for any route path, since a direct page load/refresh on `/users/42` is a genuine HTTP request the server doesn't recognize unless configured to fall back to the SPA's entry point.

## Do we only need `fetch()` instead of Axios?

**What `fetch()` provides natively now:** Promise-based/async-await compatible, `AbortController` cancellation support, streaming response bodies.

**What Axios still provides that raw `fetch()` doesn't, out of the box:**

1. **Automatic JSON handling and auto-throw on HTTP errors** — the single biggest practical difference:
```js
// Axios — built in
const { data } = await axios.get(url); // auto-parses JSON, auto-throws on 4xx/5xx
```
Matches what most developers intuitively expect (and what raw fetch doesn't do, per the gotcha above) — with fetch, the `response.ok` check must be remembered every single call or wrapped once yourself.

2. **Interceptors** — centralizing auth token attachment or global 401-redirect logic in one place:
```js
axios.interceptors.request.use(config => { config.headers.Authorization = `Bearer ${getToken()}`; return config; });
axios.interceptors.response.use(res => res, error => { if (error.response?.status === 401) router.push('/login'); return Promise.reject(error); });
```
No built-in fetch equivalent — would need a hand-built wrapper.

3. Automatic request/response transformation, simpler cancellation API, upload/download progress tracking, built-in XSRF token handling — all provided directly by Axios, requiring extra work or separate small libraries with raw fetch.

4. **Timeout handling** — fetch has none built in; requires manually wiring `AbortController` + `setTimeout`, versus a simple config option in Axios.

**Honest practical answer:**
- **If a project already uses Axios** (genuinely likely given a Quasar/Vue/Pinia stack, where Axios has been the long-standing ecosystem default) — no real reason to migrate away; the interceptor pattern is genuinely valuable for centralizing auth/error handling across many API calls in a real dashboard app.
- **For a brand-new, zero-dependency project** — raw `fetch()` is genuinely viable now, especially with a small once-written wrapper handling `response.ok`/JSON parsing/basic timeout — though the interceptor pattern still has to be built manually.
- **The fair framing:** not "fetch replaced Axios" — fetch closed the fundamental gap (Promises/async-await), but Axios still provides real, non-trivial convenience (auto-throw, interceptors, timeout) that would otherwise need to be hand-built. Whether that convenience is worth the dependency remains a legitimate, still-open tradeoff — not a settled "always use X."

## Where we left off

Second slice of Section 14 covered: `fetch()`'s critical non-throwing-on-HTTP-errors gotcha, Headers/Request/Response objects and the response-body-read-once limitation, localStorage/sessionStorage distinctions, the awkward `document.cookie` API, the History API as Vue Router's actual foundation (with the real SPA server-config implication), and a thorough, practically-grounded fetch-vs-Axios comparison tailored to the likely-Axios-already-in-use reality of the actual Quasar/Vue stack.

**Remaining Section 14 slices (planned for follow-up sessions):** IntersectionObserver/ResizeObserver/MutationObserver, requestAnimationFrame, Web Workers, Intl API.
