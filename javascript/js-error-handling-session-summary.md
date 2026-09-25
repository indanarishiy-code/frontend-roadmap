# Session Summary: Error Handling (JS Section 11)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## try/catch/finally basics

```js
try {
  riskyOperation();
} catch (error) {
  console.error(error.message);
} finally {
  cleanup(); // ALWAYS runs — try succeeds, throws, or catch itself throws
}
```

## `finally` and `return` — the mechanism explained precisely

```js
function test() {
  try {
    return "try value";
  } finally {
    console.log("finally runs"); // logs BEFORE the function actually returns
  }
}
```

**The key distinction: "deciding to return" vs "actually returning."** When `try` hits `return`, the function doesn't immediately exit — the engine computes the value and remembers it as "the value that will eventually be returned," but pauses before actually handing control back. It checks for a `finally` block and runs it first. Only after `finally` completes does the function actually complete the return.

**Why designed this way:** `finally`'s entire purpose is guaranteeing cleanup runs no matter how the try block exits — normal completion, thrown error, or `return`. If `finally` skipped whenever `return` was used, it would be nearly useless for real cleanup scenarios, since `return` is one of the most common ways a function exits:
```js
function fetchData() {
  showLoadingSpinner();
  try {
    return getData();      // early return on success
  } catch (error) {
    return null;             // early return on failure
  } finally {
    hideLoadingSpinner();     // MUST run regardless of which path was taken
  }
}
```

**The dangerous gotcha — `finally` can silently override a pending return:**
```js
function bad() {
  try {
    return "try value";
  } finally {
    return "finally value"; // this WINS — try's return is silently discarded
  }
}
```
Since the return hasn't actually completed when `finally` runs (it's still "pending"), a `return` inside `finally` simply replaces the pending one — same "return is paused until finally finishes" mechanism, not a special-case exception. **Anti-pattern:** never put `return` inside `finally` — use `finally` only for cleanup side effects, never to control what a function returns.

## Custom error classes

```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);            // calls Error's constructor, sets this.message
    this.name = "ValidationError"; // otherwise inherited name is just "Error"
    this.field = field;          // custom, additional data
  }
}
throw new ValidationError("Email is required", "email");
```
Builds directly on `extends`/`super` from Section 4 — inherits `message`/`stack`/general error behavior, adds domain-specific data.

**Why setting `this.name` matters:** without it, `error.name` is just `"Error"` for every custom class, defeating the purpose of distinct error types. `error.name` and `error instanceof ValidationError` are both genuinely useful for catching code to distinguish error types — skipping `name` silently breaks the first check.

```js
try {
  throw new ValidationError("Email is required", "email");
} catch (error) {
  if (error instanceof ValidationError) { highlightField(error.field); }
  else { showGenericError(); }
}
```
`instanceof` here works precisely because `ValidationError.prototype`'s chain includes `Error.prototype` — directly ties back to the prototype chain deep dive.

## Built-in error types

```js
new TypeError("x is not a function");     // wrong type used
new RangeError("Invalid array length");    // value outside allowed range
new ReferenceError("x is not defined");     // referencing something that doesn't exist
new SyntaxError("Unexpected token");         // malformed code, usually thrown by the parser itself
```
Rarely constructed directly — mostly thrown automatically by the engine. Recognizing which type appears in a stack trace is genuinely useful debugging info (a `TypeError` immediately narrows the search to "something's the wrong type").

## Error handling in async code — real, common pitfall

```js
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user:", error);
    throw error;
  }
}
```

**The genuine common bug — missing `await`:**
```js
async function bad() {
  try {
    fetch('/api/data'); // MISSING await — returns a Promise immediately, catch never sees a rejection
  } catch (error) {
    console.error(error); // never runs, even if the fetch ultimately fails
  }
}
```
Without `await`, `try/catch` has already exited by the time the Promise actually settles — the error happens "later," outside the synchronous try/catch block's window. A common real async bug, purely a missing-keyword mistake, easy to introduce accidentally in a refactor.

## `using` declarations — genuinely new, low current practical relevance

```js
function readFile() {
  using file = openFile("data.txt"); // hypothetical API with Symbol.dispose support
  const contents = file.read();
  return contents;
} // file automatically, deterministically "disposed" here, even if an error was thrown
```
**Problem solved:** manually remembering cleanup in `finally` (closing files, releasing locks, disconnecting DB connections) is easy to forget and messy with multiple ordered resources. `using` + the new `Symbol.dispose` protocol lets a resource define its own cleanup once, automatically invoked when the variable goes out of scope — no `finally` needed.

**Practical relevance:** genuinely low right now — requires the resource/library/API itself to implement `Symbol.dispose`, still rare in the ecosystem. Know the concept (deterministic automatic cleanup) rather than expecting to use it in typical Vue/dashboard work yet.

## Where we left off

Section 11 (Error Handling) covered in full, with real depth on the `finally`/`return` interaction (including why the language is designed this way, not just what happens) and the missing-`await` async error-handling pitfall — both genuinely common, real bug sources worth having solid.

Next up per the JavaScript reference doc: **Section 12 — Scope & Execution Context** (global/function/block scope, lexical scoping, execution context and the call stack, hoisting, strict mode) — largely already covered in depth through the earlier var/let/const, hoisting, TDZ, and closures sessions.
