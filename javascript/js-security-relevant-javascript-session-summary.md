# Session Summary: Security-Relevant JavaScript (Section 22)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Cross-Site Scripting (XSS) — the core mechanism

XSS happens when untrusted data gets executed as code instead of treated as inert text.

```js
// DANGEROUS — attacker-controlled string executed as HTML
element.innerHTML = userComment; // e.g. "<img src=x onerror='steal(document.cookie)'>"

// SAFE — always treated as plain text, never parsed as markup
element.textContent = userComment;
```

**Where JS contributes:** any API that parses a string as HTML (`innerHTML`, `outerHTML`, `document.write()`, `insertAdjacentHTML()`) is a potential injection point if the string includes anything attacker-influenced.

**Where JS/frameworks prevent it:** `textContent`, and framework templating (Vue's `{{ }}` interpolation, React's JSX children) escape by default. This is exactly why Vue's `v-html` directive is a distinct, deliberately-named escape hatch — it signals "intentionally bypassing automatic escaping here," so it should only ever be used on trusted or already-sanitized content.

## `eval()` and `Function()` constructor risks

```js
eval(userInput);              // executes ANY string as JS
new Function(userInput)();    // functionally equivalent risk, slightly different scoping
```
Both execute a string as live JavaScript. If that string can be influenced by user input — even indirectly (a config value, URL parameter, stored data) — it's a direct arbitrary-code-execution vector, not just a style-preference "best practice to avoid." Legitimate uses in application code are extremely rare (some templating/sandboxing libraries use them internally, deliberately, with their own controls).

## Same-Origin Policy (SOP) and CORS

**SOP** is the browser's default rule: a page can only freely read responses from the same origin (scheme + host + port) it was loaded from. This is browser-enforced, not server-enforced — it protects a user's data on one site from being read by JS on another site they happen to have open.

**CORS** is the mechanism a server uses to deliberately relax SOP for specific cross-origin callers:
```
Access-Control-Allow-Origin: https://trusted-app.example.com
```
When a frontend calls `fetch()` on a different origin than the API, the browser sends the request but blocks the JS from *reading* the response unless the server's CORS headers explicitly permit that origin. This is why a misconfigured backend can produce a CORS error in the browser console even though the request succeeded server-side — the block happens on the read, in the browser, not on the network call itself.

## Sanitizing user input before DOM insertion

When genuinely rendering user-supplied HTML (rich-text comment, markdown preview), use a dedicated sanitization library rather than hand-rolled regex stripping:
```js
import DOMPurify from "dompurify";
element.innerHTML = DOMPurify.sanitize(userHtml); // strips dangerous tags/attributes, keeps safe markup
```
**Rule of thumb:** if you're about to write your own "strip `<script>` tags" logic, that's the signal to reach for a maintained sanitizer instead — attackers have many ways to smuggle executable content past a naive blocklist (event handler attributes, `javascript:` URLs, malformed tags that still parse as elements in real browsers).

## Where we left off

Section 22 closed at senior-frontend depth: the XSS/`innerHTML` mechanism and why framework escaping (and Vue's `v-html` as a deliberate opt-out) exists, `eval()`/`Function()` as real arbitrary-code-execution vectors, the SOP-vs-CORS distinction (browser-side block, not a network-level one), and sanitization via a maintained library rather than hand-rolled stripping. No exercise built this round — offered but skipped in favor of summarizing.

**This closes out the entire JavaScript — Complete Learning Reference document (Sections 1–22).** Next in the personal roadmap: the TypeScript reference doc.
