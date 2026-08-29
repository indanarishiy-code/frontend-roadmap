# Session Summary: Void Elements, Comments, Whitespace, Case Sensitivity

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Void elements

Elements that can never have children/content, so they have no closing tag:
`<img>`, `<br>`, `<hr>`, `<input>`, `<meta>`, `<link>`, `<area>`, `<base>`, `<col>`, `<embed>`, `<source>`, `<track>`, `<wbr>`

```html
<img src="cat.jpg" alt="A cat">
<br>
<input type="text">
```

- Never written as `<img></img>` or `<br></br>` — nothing to close
- The trailing slash (`<img />`) is a stylistic holdover from XHTML — optional in HTML5, has zero effect on parsing. `<img>` and `<img />` produce the identical DOM node

## Comments

```html
<!-- This is a comment -->
```

- Not rendered, not exposed to screen readers, but fully visible in page source — never put sensitive info in them
- Cannot be nested
- The `--` sequence isn't allowed inside comment content per spec (though browsers are lenient about it in practice)

## Whitespace handling

- The parser collapses runs of whitespace (spaces, tabs, newlines) into a single space when rendering
- This is why source indentation doesn't create visible gaps:
```html
<p>Hello
        world</p>
```
renders as `Hello world`
- Exceptions: `<pre>` preserves whitespace exactly as written; CSS `white-space: pre` / `pre-wrap` can override default collapsing on any element

## Case sensitivity

- HTML element and attribute **names** are case-insensitive: `<DIV>`, `<Div>`, `<div>` all parse identically; `SRC` = `src`
- Different from XML/XHTML, which was strictly case-sensitive
- Convention is all-lowercase — expected by style guides, linters, formatters, and matches internal DOM normalization (`element.tagName` returns uppercase for HTML elements — a DOM quirk, not a reason to write source in uppercase)
- **Case DOES matter for attribute values that are inherently case-sensitive** — e.g. `class="Foo"` vs `class="foo"` are different values, since CSS class matching is case-sensitive

## Where we left off

This slice (void elements, comments, whitespace, case sensitivity) is closed — covered directly without extra questions needed. Remaining Section 1 slices: character encoding (`charset`) & entity references, and `<head>` contents (`<title>`, `<base>`, `<meta>` variants, `<link>` rel types, script loading `async`/`defer`/`type=module`).
