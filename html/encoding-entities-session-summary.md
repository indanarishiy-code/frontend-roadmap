# Session Summary: Character Encoding & Entity References

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Character encoding — the `charset` declaration

```html
<meta charset="UTF-8">
```

- HTML files on disk are just bytes; encoding determines how those bytes map to characters
- UTF-8 is the modern universal standard — represents every character in every language (emoji, Chinese, Arabic, accented Latin, etc.) via a variable-length byte scheme
- Without a correct `charset` declaration, the browser guesses the encoding — a wrong guess produces garbled text ("mojibake"), e.g. `Ã©` appearing instead of `é`
- **Rule:** `<meta charset="UTF-8">` must be the first thing inside `<head>`, within the first 1024 bytes of the file — the browser needs to detect encoding before parsing anything else correctly

## Entity references

Used to insert characters that conflict with HTML syntax, or are hard to type directly.

**Named entities (the ones actually worth remembering):**
```html
&lt;    →  <
&gt;    →  >
&amp;   →  &
&nbsp;  →  (non-breaking space)
```
- `&lt;`/`&gt;` needed because literal `<`/`>` in text risks being misread as tag syntax
- `&amp;` needed because `&` starts entity syntax itself
- `&nbsp;` needed because a literal space collapses per normal whitespace rules; this doesn't

**Numeric references** — for characters without a common name:
```html
&#169;   (decimal)
&#xA9;   (hex)
```
With UTF-8 properly declared, typing the actual character directly (©, é, 中) usually works fine instead — entities are mainly necessary for the syntax-conflicting characters and `&nbsp;`, not general typing convenience anymore.

## Practical takeaway (why this feels abstract without direct experience)

- Frameworks/build tools (Vite, Next.js, etc.) default to UTF-8 output automatically — this is invisible 99% of the time, which is why it's easy to forget
- **The one bug this explains:** opening a file where "café" displays as "cafÃ©" — that's an encoding mismatch, usually a file saved in one encoding but declared/read as another (e.g. missing `<meta charset>`, or a legacy file with different origin encoding)
- **Reduced practical checklist:**
  1. `<meta charset="UTF-8">` should be the first line in `<head>` — just confirm it's there
  2. If mojibake appears (`Ã©`, `â€™`, etc.) — that's the signal to check this exact line
- Depth calibration: recognizing the symptom + knowing the one-line fix is sufficient; the byte-level "why" isn't something that needs to be memorized without direct debugging experience

## Where we left off

Character encoding & entity references slice closed at a "recognize symptom + fix" level. Remaining Section 1 slice: `<head>` contents (`<title>`, `<base>`, `<meta>` variants, `<link>` rel types, script loading `async`/`defer`/`type=module`) — the last one in Section 1.
