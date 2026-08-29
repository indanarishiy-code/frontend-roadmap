# Session Summary: Doctype & Quirks Mode

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## What we covered

**Why `<!DOCTYPE html>` is needed**
- It's a signal that tells the browser which rendering mode to use, not a "version declaration"
- No doctype / malformed doctype → **Quirks Mode**: browser emulates old 1990s rendering behavior (different box model math, weird table sizing, inconsistent CSS handling)
- `<!DOCTYPE html>` present and correct → **Standards Mode**: modern, spec-compliant rendering

**Why it looks the way it does**
- Old doctypes pointed to a DTD (Document Type Definition) and were long, e.g. HTML 4.01's versioned reference to a schema URL
- HTML5 / WHATWG Living Standard has no more versioned releases — it's one continuously updated spec
- So the doctype was simplified to just be a mode-switch trigger: `<!DOCTYPE html>`

**Is it still needed today?**
- Yes — permanently. This isn't legacy cruft; it's still checked on every page load by every browser
- Frameworks (Next.js, Vue, Astro, etc.) auto-generate it in base templates for this exact reason
- Quick way to verify: run `document.compatMode` in DevTools console
  - `"CSS1Compat"` = standards mode
  - `"BackCompat"` = quirks mode

**Do I need to worry about encountering quirks mode / old HTML in practice?**
- Rare for greenfield projects, but worth recognizing
- Common triggers: inherited legacy codebases, old CMS-generated pages, broken/missing doctype, doctype not placed as the very first line
- HTML email templates are a separate, related "stuck with legacy behavior" problem — not quirks mode per se, but similar constraints
- Verdict: a "recognize it when you see it" diagnostic skill, not something to deep-study — not a topic interviewers in DE/NL/MY/UAE markets will grill on for new-project work

## Exercise tried
- Ran `document.compatMode` in DevTools on a live site to confirm standards mode detection

## Where we left off
Doctype/versioning slice (Section 1 of HTML reference) is closed out. Next slice not yet chosen.
