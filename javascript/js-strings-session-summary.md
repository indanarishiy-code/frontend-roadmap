# Session Summary: Strings (JS Section 7)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Template literals were already covered in JS Section 1.*

## Common string methods

```js
"  hello  ".trim();               // "hello"
"hello".padStart(8, "0");           // "000hello"
"hello world".replace("o", "0");     // "hell0 world" — only FIRST match
"hello world".replaceAll("o", "0"); // "hell0 w0rld" — all matches
"hello".slice(1, 3);                 // "el" — supports negative indices
"hello".substring(1, 3);             // "el" — does NOT support negative, swaps args if reversed
```

**`slice()` vs `substring()`:** `slice()` supports negative indices (`slice(-3)` = last 3 characters); `substring()` treats negatives as 0 and auto-swaps arguments if start > end. A real historical quirk, not a defensible design choice. **Practical rule:** default to `slice()` — more predictable, and works identically on arrays too, giving one mental model for both.

**`replace()` vs `replaceAll()`:** `replace()` with a plain string only replaces the first occurrence — need `replaceAll()` for every occurrence. Older workaround before `replaceAll()` existed was `replace(/o/g, "0")` (regex with global flag) — still seen in older code, but `replaceAll()` is now the direct choice for simple replacements.

## `padStart()`/`padEnd()` — the parameter is TARGET LENGTH, not padding amount

```js
"hello".padStart(8, "0"); // "000hello"
```
`padStart(targetLength, padString)` pads until the string reaches `targetLength` TOTAL — not adding `targetLength` worth of new characters. `"hello"` is already 5 characters; needs only 3 more (`8 - 5`) to reach the target of 8.

```js
"hello".padStart(8, "0").length; // 8 — the TOTAL length
"hello".padStart(5, "0");           // "hello" — already 5, nothing added
"hello".padStart(3, "0");           // "hello" — already longer than target, nothing removed
"hi".padStart(8, "0");               // "000000hi" — needed 6 characters this time
```
Never truncates if the string is already at/past target length; padding amount automatically adjusts based on distance from target.

**Standard real-world use — fixed-width formatted display:**
```js
const orderId = 42;
`Order #${String(orderId).padStart(5, "0")}`; // "Order #00042"
```
Ensures consistent total displayed length regardless of actual digit count — exactly why the parameter represents target length, not padding amount.

## `String.raw` — practical use case

```js
`Line1\nLine2`;           // actual newline
String.raw`Line1\nLine2`; // literal text: Line1\nLine2, backslash-n as characters
```
A tagged template (Section 1 concept) giving raw, unprocessed string content — escape sequences left literal instead of interpreted. **Real use case:** file paths (`String.raw\`C:\Users\name\``) or regex patterns, where backslashes should be literal without manually doubling every one.

## Unicode handling — `isWellFormed()`/`toWellFormed()`

JS strings are internally UTF-16 — possible to construct an invalid "lone surrogate" (half of a paired unit for certain emoji/special characters), from malformed user input, corrupted data, or encoding bugs.

```js
const malformed = "abc\uD800";
malformed.isWellFormed();  // false
malformed.toWellFormed();   // "abc\uFFFD" — invalid part replaced with a replacement character
```
**Practical relevance:** matters when sending strings to APIs that reject malformed Unicode, or when user-generated content (chat messages, form input) may have been corrupted in a data pipeline. `toWellFormed()` is the clean native sanitization method, replacing manual byte-level workarounds.

## Regular expressions — core syntax

```js
const regex = /hello/i;                // i = case-insensitive
"cat, hat, bat".match(/.at/g);            // g = global — ["cat", "hat", "bat"]
/^\d+$/.test("12345");                     // true
```

**Named capture groups — more readable than positional:**
```js
const match = "2026-01-15".match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/);
match.groups.year;  // "2026"
```
Access matches by meaningful name instead of fragile positional index — default to this over positional groups for anything beyond a trivial one-group pattern.

**Lookahead/lookbehind — match based on context without including it:**
```js
"$100".match(/(?<=\$)\d+/);  // ["100"] — lookbehind, digits preceded by $, $ excluded from result
"100px".match(/\d+(?=px)/);   // ["100"] — lookahead, digits followed by px, px excluded
```
Useful for extracting values based on context (currency symbols, units) without a capture group plus manual stripping.

**The `v` flag (ES2024):** extends Unicode property matching with set operations (subtraction, intersection) inside character classes. Genuinely niche/advanced — not something typical form-validation regexes need.

## `RegExp.escape()` — ES2025, solves a real common problem

```js
const userInput = "1 + 1 = 2?";
const escaped = RegExp.escape(userInput); // "1 \+ 1 = 2\?"
new RegExp(escaped).test("1 + 1 = 2?");     // true
```
**Problem solved:** building a regex dynamically from user input (a search feature matching exact typed text) breaks on special regex characters (`+`, `?`, `.`, `*`) if inserted raw. Before this, escaping required a manual regex-of-regexes function most people copy-pasted rather than wrote themselves. Now the native, correct way to safely treat arbitrary text as a literal string within a regex — genuinely practical for "search within text" features in a dashboard.

## Where we left off

Section 7 (Strings) covered in full, with a dedicated clarification on `padStart()`'s target-length (not padding-amount) parameter semantics. `slice()` vs `substring()`, named capture groups, and `RegExp.escape()` flagged as the most practically useful pieces for real dashboard work involving search/filtering/text processing.

Next up per the JavaScript reference doc: **Section 8 — Numbers, Math & BigInt** (floating point precision issues, `parseFloat`/`parseInt`/`isInteger`/`isNaN`/`toFixed`, Math object methods, BigInt, Float16Array and typed array precision).
