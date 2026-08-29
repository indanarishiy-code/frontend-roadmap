# Session Summary: HTML Tree Construction

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What tree construction is

- The stage after tokenization
- Consumes the flat token stream and builds the actual DOM tree
- Uses two pieces of state:
  1. **Stack of open elements** — tracks which elements are currently open, in nesting order (like a call stack for tags)
  2. **Insertion mode** — the parser's current "mode of behavior" (`initial`, `before html`, `in head`, `in body`, `in table`, etc.), each with its own rules for handling tokens

## Core mechanic

- Whatever's on top of the stack = current insertion point
- Opening a tag → create node, append to current top of stack, push it onto the stack
- Closing a tag → pop the stack back until that element is removed
- Correct nesting in source HTML → correct nesting in DOM tree, enforced mechanically by the stack

### Basic trace: `<p>Hi</p>`
```
StartTag(p)     → create <p>, append to current top, push        Stack: [html, body, p]
Character("Hi") → text node "Hi" appended to p                   Stack unchanged
EndTag(p)       → pop until <p> removed                          Stack: [html, body]
```
Result:
```
body → p → "Hi"
```

## Insertion modes — why context changes the rules

Same token can be handled differently depending on current mode:
- **`in head`** — a stray body-level tag (e.g. `<div>`) triggers implicit `<body>` insertion and switches mode to `in body`, even if `<body>` was never written
- **`in body`** — normal page content; most everyday intuition lives here
- **`in table`** — certain content isn't allowed as a direct child of `<table>`, so it gets redirected (see foster parenting below)

## Implied tags (auto-closing) — practically important

Some end tags are optional by spec; the parser auto-closes them.

### Traced example: `<ul><li>First<li>Second</ul>`
```
StartTag(ul)         → create <ul>, push                         Stack: [html, body, ul]
StartTag(li)         → create <li>, push                         Stack: [html, body, ul, li]
Character("First")   → text node appended to li
StartTag(li)         → RULE: <li> already open → implicitly close it first
                        → pop li, then create+push new <li>       Stack: [html, body, ul, li]
Character("Second")  → text node appended to new li
EndTag(ul)           → pop until <ul> removed
```
Result:
```
ul
 ├─ li → "First"
 └─ li → "Second"
```
Two separate `<li>` elements, correctly separated — even with no `</li>` written. Same auto-close logic applies to `<p>` (closes when a block element follows) and to `<td>`/`<tr>` in tables.

**Practical relevance:** explains why HTML with missing optional end tags still renders with correct structure instead of nesting incorrectly.

## Foster parenting — table-specific, explains a real bug pattern

Text (and most other content) isn't a legal direct child of `<table>`. Instead of discarding it or nesting it illegally, the parser moves it to just **before** the table in the tree.

### Traced example: `<table>stray<tr><td>Cell</td></tr></table>`
```
StartTag(table)      → create <table>, append to body, push       Stack: [html, body, table]
                        mode switches to "in table"
Character("stray")   → RULE: text not allowed directly in <table> here
                        → FOSTER PARENT: insert as child of body, right BEFORE
                          the table element (not inside it)
StartTag(tr)/(td)    → proceed normally, nested inside table
Character("Cell")    → legal here, appended to td
EndTag(td)/(tr)/(table) → pop back down normally
```
Result:
```
body
 ├─ "stray"   ← ends up BEFORE the table, not inside it
 └─ table
     └─ tr → td → "Cell"
```

Verified hands-on: tested with a real `<table>` containing stray text before `<tr>` — confirmed the stray text renders visually above/outside the table.

**Practical relevance:** explains a real recurring bug — misplaced content near a `<table>` (often from templating bugs) silently renders in the wrong place on the page instead of where it was written in source.

## Why foster parenting works this way

- **Why not allow stray content inside `<table>`?** Tables have a specialized layout algorithm with no defined behavior for loose content directly inside them. Pre-HTML5-spec, browsers handled this inconsistently — a real interoperability problem. The WHATWG spec generally standardized "what browsers already did in practice" rather than inventing new behavior, since the existing web already depended on it.
- **Why *before* the table specifically, not after?** Parsing is a streaming, incremental process. At the moment the stray text token is encountered, the parser has already created the `<table>` node and appended it to its parent — but the table isn't finished yet (its closing tag hasn't been seen). "Insert after the table" isn't a computable instruction yet, since the table's contents/end are still in the future. "Insert before the table's current position in its parent" is the only position the parser can actually resolve at that moment — so that's what foster parenting does.

## Depth calibration for senior frontend role

**Solid enough to stop at:**
- Stack + insertion mode as the two core mechanisms
- Implied/auto-closing tags
- Foster parenting around tables, and the reasoning behind why it behaves this way

**Not worth going deeper into:**
- The full list of ~20+ insertion modes and their individual token-handling tables
- The adoption agency algorithm in full step-by-step detail (misnested formatting tags, e.g. `<b>bold <i>both</b> italic</i>`) — worth knowing it exists and roughly what it's for, not worth tracing exact steps
- Table-specific edge cases beyond foster parenting

**Practical interview-level summary:** "the parser maintains a stack of open elements, uses context-dependent insertion modes, and has defined auto-recovery behavior for common malformed patterns like unclosed tags and misplaced table content."

## Where we left off

Tree construction slice closed at senior-frontend-appropriate depth. Error recovery (adoption agency algorithm) is the next planned slice, at a "know it exists, roughly what it's for" level of depth — not full step-by-step detail.
