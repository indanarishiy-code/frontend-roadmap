# Session Summary: TypeScript — Core Concepts & Why TypeScript Exists (Section 1)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What TypeScript is

A superset of JavaScript — every valid `.js` file is valid TypeScript — adding a static type layer that's checked at compile time and then fully erased before runtime (ties directly to the type-erasure point from the earlier JS "boundary awareness" session). Core value: catching a category of bugs (wrong argument types, typos in property access, unhandled `null`/`undefined`) before the code runs, rather than in production.

## Structural typing ("duck typing") — and why TypeScript uses it

Two types are compatible if their **shape** matches — no explicit inheritance or declared relationship required. Genuinely different from nominal typing (Java/C#), where compatibility depends on declared identity (`implements`, `extends`).

```ts
interface Point { x: number; y: number; }

function printPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

const coord = { x: 3, y: 5, label: "origin" }; // never declared as a Point
printPoint(coord); // valid — shape matches, extra props are fine
```

**Why TypeScript is built this way, not stricter:** TypeScript describes an *existing* language's runtime behavior — it doesn't get to redefine it. JavaScript itself is structural at runtime: any object with the right properties already works anywhere that shape is expected, with no concept of formal type membership. A nominal system would break constantly against ordinary JS patterns — a `JSON.parse()` result, a plain object literal, a merged/spread object — none of which has (or should need) a declared relationship to an interface:

```ts
const data = JSON.parse(apiResponse); // { x: 3, y: 5 } — no class, no declared relationship
printPoint(data); // needs to work — this is normal, correct JS/TS usage
```

Under nominal rules, this would fail type-checking despite being correct at runtime. Structural typing isn't "looser" where it counts — it still fully checks that every required property exists with the correct type. What it gives up is caring *how* the object came to have that shape, which is exactly what makes plain-object interop (API responses, object literals, merges) work without ceremony.

**The real cost, and where it's addressed later:** two structurally identical but conceptually different types (a `UserId` string vs. a plain `string`) are freely interchangeable by default — a real gap. This is precisely what **branded/nominal types** (Section 15 of this doc) exist to patch; noted now as the direct answer to "doesn't this get too loose sometimes," full pattern to come later.

## `tsconfig.json`

The compiler configuration file — controls target JS output version, module resolution strategy, and strictness flags (individual `strict*` flags get their own dedicated slice in Section 12). Enough for now to know it exists and roughly what categories of settings live there.

## Where we left off

Section 1 closed with strong reinforcement rather than new material — expected, since TypeScript already appears across every framework reference doc. The structural-vs-nominal distinction and its rationale were the one point that got real depth. Next: **Sections 2-4** (Basic Types, Object Types & Interfaces, Functions) — per the doc's own suggested path, moving toward Section 5 (union types and discriminated unions) as the first genuinely high-value depth section.
