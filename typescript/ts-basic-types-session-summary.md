# Session Summary: TypeScript — Basic Types (Section 2)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Primitives, arrays, tuples

Primitives (`string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`) are the same as JS, just annotated.

```ts
const names: string[] = ["a", "b"];        // array — any length, all same type
const point: [number, number] = [3, 5];    // tuple — fixed length, fixed type per position
point[2]; // TypeScript error — tuple only has 2 positions
```
`string[]` and `Array<string>` are equivalent syntax — bracket form more common day-to-day, generic form more common in library/utility-type contexts.

## `any` vs `unknown`

```ts
let a: any = fetchSomething();
a.doAnything().whatever(); // compiles fine — `any` opts OUT of type checking entirely, and downstream too

let u: unknown = fetchSomething();
u.doAnything(); // ERROR — must narrow first
if (typeof u === "object" && u !== null && "doAnything" in u) {
  (u as { doAnything: () => void }).doAnything(); // now safe
}
```
`any` is a hole in the type system — once a value is `any`, checking stops for it and everything downstream that touches it, which is why overuse is a real code smell, not a style nitpick. `unknown` is the type-safe version of "not known yet" — forces proving the type (via `typeof`, `instanceof`, a custom guard) before use. Prefer `unknown` at genuine unknowns (API responses, `JSON.parse()`, catch-block errors); treat `any` as something to actively eliminate.

## `void` vs `never`

```ts
function log(msg: string): void { console.log(msg); }        // doesn't return a useful value
function fail(msg: string): never { throw new Error(msg); }  // never returns at all
```
`never` becomes genuinely useful later for exhaustiveness checking with discriminated unions (Section 15).

### Why `void` isn't just typed as `undefined`

Not about runtime return value — about **assignability**, specifically for callbacks:
```ts
function callWithCallback(cb: () => void) { cb(); }
callWithCallback(() => 42); // WORKS — void means "return value is ignored," any function qualifies

function callWithCallback2(cb: () => undefined) { cb(); }
callWithCallback2(() => 42); // TYPE ERROR — undefined return type is a strict promise
```
`void` deliberately means "I don't care what you return" — this is why array-method callbacks (`forEach`, etc.) are typed to return `void`: it lets an arrow function with an implicit return (`(x) => x * 2`) be passed even though it technically returns a value, because the caller discards it. Typing such callbacks as `undefined` instead would break this extremely common pattern for no real benefit.

## `Object` (capital O) vs `Array<string>`

Flagged that `Object<string[]>` isn't valid TypeScript syntax — `Object` isn't generic, so it can't take a type parameter.
```ts
let a: Object = { anything: "goes", could: 42, be: [1,2,3] }; // extremely loose, barely checked
let b: Array<string> = ["x", "y"]; // precise — array of strings specifically
```
`Object` (capital) means "any non-null, non-undefined value" — arrays, functions, boxed numbers, everything — and gives almost no real safety; modern TypeScript reaches for lowercase `object` ("not a primitive") or a specific shape instead. `Array<string>` is precise and exactly equivalent to `string[]`. The two aren't really comparable alternatives — `Array<string>` is a real, useful type; `Object` is a near-useless catch-all rarely reached for deliberately.

## Type inference

```ts
let count = 5; // inferred as `number`, no annotation needed
```
Let inference handle local variables and simple returns; add explicit annotations at function boundaries (parameters, and often return types on exported/public functions) — that's where the type is an actual contract other code depends on.

## Where we left off

Section 2 closed — most of it reinforcement, with real depth added on `any`/`unknown`/`void` mechanics and the `Object`-vs-`Array<string>` clarification. Next: **Sections 3-4** (Object Types & Interfaces, Functions), continuing toward Section 5 (union types and discriminated unions) as the next major depth section.
