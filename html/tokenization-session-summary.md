# Session Summary: HTML Tokenization

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What tokenization is

- The first stage of browser HTML parsing
- Reads raw HTML text character by character
- Groups characters into **tokens** — labeled chunks of data
- Does NOT build any structure/tree — it doesn't know what's nested inside what, it just outputs a flat, ordered stream

## Token types
- Start tag token — e.g. `<p>`
- End tag token — e.g. `</p>`
- Character token — plain text
- Comment token
- DOCTYPE token

## Basic example

Input:
```html
<p>Hi</p>
```

Tokenizer output (flat stream, no nesting info):
```
StartTag(p)
Character("Hi")
EndTag(p)
```

Building the actual tree (knowing `<p>` contains this text, and what it's nested inside) is the **next** stage — tree construction — not tokenization's job.

## Special tokenizer states (practically relevant)

Most content is tokenized in the default **Data state**. But some elements switch the tokenizer into different states because their content shouldn't be parsed as normal markup:

- **RAWTEXT state** — `<style>`: no tags or entities recognized inside; everything until the matching end tag is literal text
- **Script data state** — `<script>`: similar to RAWTEXT, so `<` inside JS code (e.g. `if (x < 3)`) doesn't get misread as a tag open
- **RCDATA state** — `<textarea>`, `<title>`: character references (`&amp;`) still decode, but tags do NOT — so typing `<div>` inside a `<textarea>` renders as literal text, not a nested element

**Practical relevance:** explains why you can't naively embed `</script>` inside inline JS strings, and why `<textarea>` can safely contain markup-looking text.

## Character reference decoding

- Happens during tokenization, not later
- `&amp;`, `&lt;`, `&#169;` (decimal), `&#xA9;` (hex) all get decoded into their literal characters at this stage
- By the time tree construction sees the token, the decoding has already happened
- Practical relevance: matters when building HTML strings dynamically — `&amp;` vs `&` distinction

## Depth calibration for senior frontend role

**Worth knowing solidly:**
- Tokenization exists as a distinct first stage, produces a flat token stream
- The 3 special-content elements (`<script>`, `<style>`, `<textarea>`) and why they're tokenized differently
- Character references decode during tokenization

**Below the useful line for frontend work (browser-engine-level detail, not covered further here):**
- The ~80 named tokenizer states
- The "reconsume" lookahead/backtracking mechanism
- EOF-mid-token edge case handling
- Ambiguous-ampersand legacy quirks (e.g. `&copy` without semicolon)

**Where to focus depth instead:** tree construction and error recovery (adoption agency algorithm, implied tags, foster parenting) — these explain real "why did my HTML render weird" bugs and come up more in interviews.

## Where we left off

Tokenization slice closed at an appropriate depth for a senior frontend role. Tree construction is the next planned slice (not yet started in depth — only briefly introduced: stack of open elements + insertion modes).
