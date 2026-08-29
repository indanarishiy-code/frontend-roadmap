# Session Summary: Text Content (Section 3)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## `<p>`

A paragraph. Can only contain phrasing content (text + inline elements) — no block-level elements (`<div>`, `<ul>`, another `<p>`) nested inside. The parser auto-closes a `<p>` if a block element appears where nesting was attempted (ties back to the tree construction "implied tags" mechanism covered earlier).

## `<blockquote>`

For quoting an extended passage from another source:
```html
<blockquote cite="https://example.com/source">
  <p>The full quoted text goes here.</p>
  <footer>— Author Name</footer>
</blockquote>
```
`cite` attribute = source URL (machine-readable, not displayed). Visible author attribution goes outside the blockquote, often via `<footer>`.

## `<pre>`

Preserves whitespace exactly as written — no collapsing. Used for code blocks, ASCII art, formatting-sensitive content. Commonly paired with `<code>`: `<pre><code>...</code></pre>`.

## `<hr>`

A thematic break — signals a shift in topic, not just a decorative line. Void element, no closing tag.

## Lists

**Unordered** (`<ul>`) — no inherent sequence.
**Ordered** (`<ol>`) — sequence matters. Supports `start` and `reversed` attributes; each `<li>` can have a `value` attribute to override its number.

**Description list** (`<dl>`/`<dt>`/`<dd>`) — key/value or term/definition pairs, useful beyond dictionaries (metadata displays, FAQs, spec sheets):
```html
<dl>
  <dt>HTML</dt>
  <dd>HyperText Markup Language</dd>
</dl>
```

**`<dl>` flexibility (not fixed to one `<dt>`/one `<dd>`):**
- Multiple `<dt>`s can share one `<dd>` (synonyms/abbreviations mapping to one definition)
- One `<dt>` can have multiple `<dd>`s (multiple definitions/descriptions for one term)
- A `<dl>` can nest inside a `<dd>` for genuine hierarchical data (rare in practice — most real-world `<dl>` use is flat, one pair per row)
- Core rule: every `<dt>` must be followed by at least one `<dd>` (or another `<dt>`) — no arbitrary depth limit

## Inline text semantics — the classic visual-vs-semantic pairs (most important part)

**`<em>` vs `<i>`** — both render italic:
- `<em>` = emphasis, changes sentence meaning/stress when read aloud
- `<i>` = alternate voice/mood, no added emphasis (technical terms, foreign phrases, thoughts, ship names)
```html
<p>You <em>must</em> submit this by Friday.</p>  <!-- emphasis -->
<p>The term <i>hoisting</i> refers to...</p>      <!-- alternate voice -->
```

**`<strong>` vs `<b>`** — both render bold:
- `<strong>` = strong importance/urgency/seriousness
- `<b>` = stylistically bold, no added importance (keywords, product names)
```html
<p><strong>Warning:</strong> deleting this cannot be undone.</p>
<p>The recipe needs <b>flour</b>, <b>sugar</b>, and <b>eggs</b>.</p>
```

**Why this matters practically:** screen readers treat `<em>`/`<strong>` as meaningful and may change vocal inflection/emphasis; `<i>`/`<b>` carry zero semantic signal to assistive tech — purely visual. `<u>` behaves similarly to `<i>`/`<b>` (visual only) and is generally discouraged since underline visually implies "link."

**This is the single most interview-relevant distinction in this whole batch.**

## Citation, quoting, definitions

- `` — title of a creative work (book, movie, song), not the author: `<cite>The Great Gatsby</cite>`
- `<q>` — short inline quote; browsers auto-add quotation marks
- `<dfn>` — marks a term at its first/defining use

## Time and machine-readable data

- `<time datetime="2026-08-29">August 29, 2026</time>` — `datetime` gives a machine-readable value while visible text stays human-friendly
- `<data value="482">Product #482</data>` — same idea for non-time values

## Code-related

- `<code>` — inline code snippet
- `<var>` — a variable name in math/programming context
- `<samp>` — sample program output
- `<kbd>` — keyboard input

## Abbreviations and small print

- `<abbr title="HyperText Markup Language">HTML</abbr>` — `title` gives the expansion, often a hover tooltip
- `<small>` — fine print, legal disclaimers, side comments — genuinely semantic, not just "smaller font"

## Miscellaneous

- `<mark>` — highlighted/relevant text (e.g. search match)
- `<sub>`/`<sup>` — subscript/superscript
- `<s>` — strikethrough for content no longer accurate/relevant (e.g. old price) — different from `<del>`, which marks edited-out content in a revision-tracking context
- `<span>` — inline equivalent of `<div>`: no semantic meaning, pure styling/JS hook
- `<br>` — line break within text (not for spacing between elements — that's CSS's job)
- `<wbr>` — suggests an optional line-break point inside a long word/URL, without forcing one

## Where we left off

Section 3 (Text Content) conceptually covered in full: block-level elements (`<p>`, `<blockquote>`, `<pre>`, `<hr>`), lists (including `<dl>` flexibility), and the full inline text semantics batch. The `<em>`/`<strong>` vs `<i>`/`<b>` accessibility distinction is flagged as the single most important takeaway from this section. Hands-on verification with a screen reader (VoiceOver on macOS) was attempted but not yet completed — worth revisiting on your own once VoiceOver is successfully enabled via System Settings → Accessibility → VoiceOver.

Next up per the HTML reference doc: **Section 4 — Links & Navigation** (`<a>` attributes including `href`/`target`/`rel`/`download`/`ping`, URL schemes, fragment identifiers).
