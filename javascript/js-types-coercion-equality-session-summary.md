# Session Summary: Data Types, Coercion, `==` vs `===` (JS Section 1, Part 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Primitives vs objects — copy by value vs reference

```js
let a = 5;
let b = a;
b = 10;
console.log(a); // 5, unaffected — primitives copied by value

let obj1 = { x: 1 };
let obj2 = obj1;
obj2.x = 10;
console.log(obj1.x); // 10, changed — objects copied by reference
```
Assigning a primitive copies the actual value. Assigning an object copies a reference to the same underlying object — `obj1` and `obj2` point at the same thing. Same mechanic behind why `const arr = []; arr.push(1)` is legal: not reassigning the reference, mutating what it points to.

## `typeof null` is `"object"` — verified historical reason (via search)

In JavaScript's original 1995 implementation, every value was stored as a **type tag** plus data. The type tag for objects was 0. `null` was represented as a literal NULL pointer (0x00), which happened to carry that same 0 type tag as objects — so `typeof` just read the tag and reported "object," with no special-case logic for null at all.

**Never fixed:** a fix was proposed for ECMAScript but rejected — it broke too much existing code that had come to depend on `typeof null === "object"`, even unintentionally. Backward compatibility won out over correctness. One of the most commonly cited JS interview "gotchas."

## Type coercion — implicit vs explicit

```js
String(123);    // "123" — explicit
Number("123");   // 123 — explicit
"5" + 3;         // "53" — implicit, + sees a string, coerces number to string
"5" - 3;         // 2 — implicit, minus has no string meaning, coerces to number
```
`+`'s dual behavior (concatenation vs addition) is the single biggest source of implicit coercion confusion — the only arithmetic operator with string meaning. `-`, `*`, `/` always coerce toward numbers.

## `==` vs `===` — the correct one-sentence framing

**`===` compares value and type with no conversion at all. `==` first tries to convert the two operands to a common type using specific spec-defined rules, and only then compares them.**

**Correction to a common oversimplification:** "`==` doesn't check type, `===` does" is imprecise. More accurate: `===` refuses to compare across types at all (different types = automatically `false`). `==` is willing to convert one or both operands to make a cross-type comparison possible, then compares — it's not "ignoring type," it's "actively resolving a type mismatch via defined conversion rules first." If both operands are already the same type, `==` does zero conversion and behaves identically to `===`.

## `"" == "0"` — worked through precisely

```js
"" == "0";  // false
```
Both operands are already strings — the `==` algorithm's first check is "are both operands already the same type?" If yes, **no coercion happens at all**, and it's a plain direct string comparison (same as `===` would give). `""` and `"0"` are just two different string values, hence `false`.

**Why people expect `true`:** both `""` and `"0"` are individually falsy when coerced to boolean, so people mentally lump them as "both falsy, therefore equal" — but `==` between two strings never invokes boolean coercion at all. Boolean coercion only applies when one side is genuinely a boolean (e.g. `0 == false`).

## `null == undefined` — the special case, and why it exists despite being conceptually different values

```js
null == undefined;  // true — a specific hardcoded special case in the spec, not general coercion math
null == false;        // false
undefined == false;   // false
null == 0;              // false
null == "";             // false
```

**The conceptual distinction is real and valid:** `null` = intentional absence ("deliberately set to nothing"); `undefined` = unintentional absence ("never set at all"). `===` fully respects this distinction (`null === undefined` is `false`).

**Why `==` treats them as equal anyway — the actual reasoning:** not a claim that they're the same thing, but a narrow, pragmatic shortcut for the common case of "does this value exist at all" — where you often don't care *which* flavor of nothing you got, only that there's nothing usable there:
```js
if (value == null) {
  // handles: never assigned (undefined) AND deliberately cleared (null)
  // in one check instead of two
}
```
This is a deliberately scoped exception for one specific common question, not a general blurring of the null/undefined distinction — which remains fully intact everywhere else in the language (`===`, `typeof`, etc.). If the distinction matters for specific logic, use `===` or check individually; if it genuinely doesn't matter, `== null` is the sanctioned shortcut. This is common enough that ESLint's `eqeqeq` rule explicitly allows this one exception while flagging every other `==` use.

`null == false` is `false` because null's special-case behavior only extends to `undefined` — null does not participate in normal numeric/boolean coercion at all.

## Practical rule for `==`/`===`

Use `===` by default, always. The one broadly accepted exception: `== null` as a deliberate idiom for catching both `null` and `undefined` in one check.

## `Object.is()` — deeper explanation, correcting a name-based misconception

**Not primarily for comparing objects, despite the name** — works on any value, primitives included. For object comparison specifically, it behaves exactly like `===` (reference comparison, adds nothing new).

```js
Object.is(NaN, NaN);   // true — === gives false, the famous NaN exception (NaN === NaN is always false per IEEE 754)
Object.is(0, -0);       // false — === gives true, but 0 and -0 are distinguishable in some contexts (e.g. division direction)
Object.is(5, 5);         // true — behaves like === for ordinary values
Object.is({}, {});        // false — same as ===, still reference comparison
```

**Actual purpose:** a more mathematically precise equality check for two narrow edge cases where `===` gives surprising results — detecting `NaN` correctly, or when the `0`/`-0` sign distinction matters (rare, mostly numeric/graphics contexts).

**Real-world relevance:** React's internal dependency comparison for some hooks uses `Object.is()`-style logic rather than `===`, specifically so a `NaN` dependency doesn't cause an infinite re-render loop (since `NaN === NaN` being false would make React think a `NaN` value "changed" every render even when it didn't).

## Where we left off

Second slice of JS Section 1 covered in real depth: primitives vs objects (copy semantics), the verified history behind `typeof null`, implicit/explicit coercion, and a thorough `==` vs `===` deep dive including the `null == undefined` special case reasoning and `Object.is()`'s actual purpose. Remaining Section 1: operators (`??`, `?.`, and the rest), truthy/falsy values, template literals/tagged templates.
