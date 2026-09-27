# Session Summary: TypeScript Context (Section 20) & Tooling/Runtime Context (Section 21)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Section 20 — TypeScript Context (boundary awareness only)

Deliberately scoped narrow — the doc itself says full TypeScript depth is a separate reference.

**What TypeScript adds:** a compile-time type-checking layer over JavaScript. No runtime behavior of its own — every `.ts` file compiles to plain JS with types stripped out entirely. At runtime, TS code behaves identically to the JS it compiles to.

**The one thing worth being solid on — type erasure:**
```ts
function greet(name: string) {
  console.log(name.toUpperCase());
}
greet(42 as any); // compiler allows it if you lie with `as any`
// Runs, crashes: 42.toUpperCase is not a function
```
There is no runtime type check happening — `string` never exists once compiled. Any boundary TypeScript can't see through (an API response, `JSON.parse()`, an untyped third-party library, an `any`) can carry a lie past the compiler. The failure then shows up as an ordinary runtime JS error, not a type error, even though the code looks fully typed:
```ts
const data = JSON.parse(rawString); // typed `any` — no safety here
data.user.name.toUpperCase(); // no compiler protection despite being "TS code"
```
**Practical takeaway:** TypeScript's safety guarantee is only as strong as the types you actually give it — the moment data crosses an untyped boundary, you're back to plain-JS runtime risk with no compiler net.

## Section 21 — Tooling & Runtime Context

### Node.js vs browser
Different global environments running the same language:
- **Browser** — `window`, `document`, DOM APIs, `fetch`, `localStorage`; no filesystem/OS access
- **Node** — `process`, `require`/ESM, `fs`; no `window`/`document`

**Real bug pattern:** a library touching `window` at import time breaks under SSR (Next.js/Nuxt) because there's no `window` on the server — the actual mechanism behind "window is not defined" SSR errors.

### `dependencies` vs `devDependencies`

Not about "useful during development" — about whether the code is **actually imported and executed by the shipped app**, versus only used by tooling that builds/lints/tests/type-checks but never ships.

```json
{
  "dependencies": { "vue": "^3.4.0", "pinia": "^2.1.0" },
  "devDependencies": { "vite": "^5.0.0", "eslint": "^9.0.0", "@types/node": "^20.0.0" }
}
```

**How to actually tell which category a package belongs in:**
- Grep your real application source for an `import` of it — if it shows up there, it's `dependencies`
- If it only appears in config files that never get bundled (`vite.config.ts`, `.eslintrc`, test files), it's `devDependencies`
- Sanity check: delete `node_modules`, run `npm install --omit=dev`, then build and run the production build — a missing-module error reveals a misclassified package

Category patterns: frameworks/runtime libs → `dependencies`; bundlers/compilers/linters/test runners → `devDependencies`; `@types/*` → always `devDependencies` (types are erased at compile time, so can never be a runtime dependency). Notably, `typescript` itself is `devDependencies` even though the whole app is written in it — the compiler never runs at runtime.

**Where misclassification bites:** `npm install --production` / `npm ci --omit=dev` skips `devDependencies` entirely (used in lean production Docker images/deploy pipelines) — a runtime-needed package wrongly placed in `devDependencies` will work fine locally but break with "module not found" in that kind of production-only install.

### `package.json` vs `package-lock.json`

`package.json` declares **intent** — a version *range*:
```json
"vue": "^3.4.0"  // any 3.x.x >= 3.4.0, npm picks the newest compatible one at install time
```
`package-lock.json` records **exactly what got installed** — the full transitive dependency tree (every dependency of every dependency), pinned to exact versions with integrity hashes:
```json
"node_modules/vue": {
  "version": "3.4.21",
  "resolved": "https://registry.npmjs.org/vue/-/vue-3.4.21.tgz",
  "integrity": "sha512-..."
}
```
**Why reproducibility needs both:** without the lockfile, two installs from the same `package.json` on different days could resolve different actual versions, since ranges only narrow possibilities rather than pin them. `npm ci` (used in CI) reads *only* the lockfile and fails if it's out of sync — the strict, reproducible install mode.

**Why they're separate files, not merged:**
- Different authorship model — `package.json` is hand-edited by you; the lockfile is 100% machine-generated/read, never hand-edited
- Wildly different size/scope — `package.json` lists ~20-30 direct deps with loose ranges; the lockfile records hundreds-to-thousands of transitive entries with exact resolved versions and hashes
- Tooling reads them for different purposes — "what did the user ask for" (`package.json`) vs "what exact tree did we resolve last time" (lockfile)
- Package managers don't even agree on lockfile format — npm's `package-lock.json`, Yarn's `yarn.lock`, pnpm's `pnpm-lock.yaml` are all different, while `package.json` is the one file all three read identically; folding lockfile data into it would break that interoperability

**Practical rule:** always commit `package-lock.json`; never hand-edit it.

### Bundlers (Webpack/Vite) — conceptual only

Core job: resolve the module graph (every `import`/`export`) and produce optimized output — tree-shaking (dropping unused exports), code-splitting (per-route chunks), HMR in dev. Vite specifically serves native ES modules directly in dev with no bundling step until build, which is why its dev server starts near-instantly compared to older Webpack setups.

### ESLint vs Prettier

ESLint catches *correctness*/*pattern* issues (unused variables, unreachable code, framework-specific rule violations). Prettier only reformats *style* (semicolons, quotes, line width). Commonly run together, with Prettier wired in as an ESLint rule (`eslint-config-prettier`) so the two don't fight over formatting opinions.

### Source maps & DevTools debugging

A source map maps minified/transpiled running code back to original source, so DevTools stack traces and breakpoints show real `.vue`/`.ts` files and lines, not compiled output. Worth being comfortable with `debugger;` statements and the Sources panel, not just `console.log`-driven debugging.

## Where we left off

Sections 20 and 21 closed — no exercises built (both are conceptual/tooling literacy, not hands-on skills; agreed neither benefits from a code exercise). Remaining in the JS reference doc: **Section 22 — Security-Relevant JavaScript** (XSS basics and how JS contributes to/prevents it, `eval()`/`Function()` constructor risks, Same-Origin Policy and CORS basics as they relate to `fetch`, sanitizing user input before DOM insertion) — the final section of the JavaScript doc.
