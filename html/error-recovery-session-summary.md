# Session Summary: HTML Parsing — Error Recovery

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What error recovery is

- The third piece of HTML parsing, alongside tokenization and tree construction
- The set of rules the parser follows whenever incoming markup doesn't strictly make sense (missing tags, tags in illegal positions, overlapping tags)
- **Core principle:** the HTML parser is specified to never fail. Every possible malformed input has exact, deterministic, defined behavior — unlike parsers (JSON, XML) that throw errors and stop
- This exists because the web has decades of already-published sloppy/buggy HTML; browsers can't refuse to render it. Before this was formally specified (~pre-2008), different browsers handled broken markup inconsistently — a real interoperability problem

## The three concrete mechanisms (the full practical taxonomy)

Error recovery isn't one mechanism — it's an umbrella term. Three specific, named rules cover the practical cases:

### 1. Implied / auto-closing tags — handles *missing* end tags
- Certain elements auto-close when another element that can't legally be their sibling/child appears
- Each element has its own defined list of "closers" — it's not "same tag closes it" universally
- Example: `<li>` closes when another `<li>` (or `<ul>`/`<ol>` boundary) appears; `<p>` closes when a block-level element like `<div>`, `<h1>`–`<h6>`, `<table>`, `<section>`, `<blockquote>`, `<form>`, etc. appears — but NOT for inline elements like `<span>` or `<strong>`, which nest normally inside `<p>`
- Traced: `<p>First<div>Second</div>` → `<div>` is on `<p>`'s auto-close list → `<p>` closes first → result is two siblings (`p` and `div`), not `div` nested inside `p`
- Compare: `<p>First<span>Second</span>` → `<span>` is NOT on the list → ends up legally nested inside `<p>`

### 2. Foster parenting — handles content in an *illegal position*
- Text (and most other content) isn't a legal direct child of `<table>`
- Instead of discarding it or nesting it illegally, the parser moves it to just **before** the table in the tree
- Traced: `<table>stray<tr><td>Cell</td></tr></table>` → "stray" ends up as a sibling of `<table>`, positioned right before it, not inside it
- **Why before, not after:** parsing is streaming/incremental. At the moment the stray text is encountered, the table node already exists and is attached to its parent, but it isn't finished (closing tag not yet seen) — so "after the table" isn't a computable position yet. "Before the table's current position" is the only position resolvable at that moment
- **Why allow this at all instead of nesting it in the table:** tables have a specialized layout algorithm with no defined behavior for loose content directly inside them; the spec generally standardized what browsers already did in practice rather than invent new behavior
- Verified hands-on: confirmed stray table text renders visually above/outside the table

### 3. Adoption agency algorithm — handles *overlapping/misnested* formatting tags

**What it is:** the specific recovery rule for when formatting elements (`<b>`, `<i>`, `<em>`, `<strong>`, `<a>`, `<s>`, `<u>`) are misnested relative to each other — meaning their closing tags don't appear in reverse order of opening.

- Correct nesting: `<b><i>text</i></b>` (i closes before b — reverse order, like parentheses)
- Misnested: `<b>bold <i>both</b> italic</i>` (b closes before i, even though b opened first)

**Why this needs special handling:** plain stack-based tree construction can't cleanly process a close tag when that element isn't on top of the stack. Naively popping everything down to it would also force-close elements still legitimately open above it, losing styling context for later content.

**What it does, at the concept level:** closes the mismatched formatting element early where the conflict happens, then clones/reopens it further along so later content keeps the same styling context instead of losing it — approximating what the author probably intended.

**Traced example:** `<b>bold <i>both</b> italic</i>`
- "bold " → bold only
- "both" → italic only (inside the original `<i>`)
- " italic" → ends up both bold AND italic — because `<b>` gets cloned and nested inside the still-open `<i>` to preserve bold styling for the remainder

**Deeper mechanics (implementer-level, beyond what's needed day to day):**
- The algorithm handles arbitrarily repeated/nested misformatting via an outer loop capped at 8 iterations (spec-hardcoded safety limit)
- An inner loop walks the stack to find the "furthest block" — the point where reparenting needs to happen
- It maintains a **bookmark** to preserve relative ordering when multiple formatting elements are cloned/reinserted at once
- It uses a **separate "list of active formatting elements"** alongside the stack of open elements — the stack tracks what's structurally open, while this list tracks styling context that may need reconstructing even after the original element closed
- This dual-state coordination with a bounded retry loop is why this is considered one of the most complex parts of the whole HTML parsing spec

## Practical takeaway (all three mechanisms)

Never rely on any of this intentionally — always write correctly nested, correctly placed tags. These mechanisms are a safety net for malformed markup that commonly comes from string-concatenated HTML, old CMS content, copy-pasted rich text, or templating bugs — not something to lean on by design.

**Recognizing the symptoms in the wild:**
- Weird/duplicate-looking tags in DevTools Elements panel → possible adoption agency recovery from overlapping formatting tags
- Content silently rendering in an unexpected position near a `<table>` → likely foster parenting
- HTML with missing optional end tags still rendering with correct structure → implied tags at work

## Depth calibration for senior frontend role

**Solid enough to stop at (this is the target level):**
- Error recovery as a concept: parser never fails, has deterministic rules for malformed input
- All three mechanisms by name, what triggers each, and one traced example each
- The adoption agency algorithm's conceptual purpose and a basic trace — not its internal loop/bookmark mechanics

**Not worth retaining/going deeper into:**
- The adoption agency algorithm's exact internal loop structure, bookmark bookkeeping, and the separate active-formatting-elements list mechanics (implementer-level detail, covered once out of curiosity but not needed for interviews or debugging)
- Insertion-mode-specific recovery rules beyond `in table` (e.g. `in select`, `in caption`)

**Practical interview-level summary:** "the HTML parser never fails on malformed input — it has defined recovery rules for missing tags (implied/auto-closing), misplaced content (foster parenting, e.g. stray table content), and misnested formatting tags (adoption agency algorithm, which reconstructs elements like `<b>`/`<i>` to preserve styling context)."

## Where we left off

This completes the full "how browsers parse HTML into the DOM" arc: **tokenization → tree construction → error recovery**. All three error recovery mechanisms (implied tags, foster parenting, adoption agency) have been covered at senior-frontend-appropriate depth, with hands-on verification for foster parenting. Ready to move to a different slice next — either back to Section 1 of the HTML reference (attributes, `<head>` contents, etc.) or elsewhere.
