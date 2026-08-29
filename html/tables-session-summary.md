# Session Summary: Tables (Section 6)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## Basic structure

```html
<table>
  <caption>Monthly Sales Report</caption>
  <thead>
    <tr>
      <th scope="col">Month</th>
      <th scope="col">Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>January</td>
      <td>$5,000</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td>Total</td>
      <td>$5,000</td>
    </tr>
  </tfoot>
</table>
```

- **`<caption>`** — table's title/description, announced by screen readers before content; should be the first child of `<table>`
- **`<thead>`/`<tbody>`/`<tfoot>`** — semantic row grouping (header/body/summary rows). Browsers may use `<thead>` to repeat header rows when a long table prints across pages — not just organizational
- **`<tr>`** — row; **`<th>`** — header cell (semantically important, not just visually bold/centered); **`<td>`** — data cell

## `scope` — the critical accessibility attribute

```html
<th scope="col">Revenue</th>   <!-- header applies to a column -->
<th scope="row">January</th>   <!-- header applies to a row -->
```

Tells screen readers which cells a header describes. Without it, a screen reader user navigating cell-by-cell has no reliable way to know what column/row they're in — `scope` is what makes a table actually navigable non-visually, not just visually organized.

## `headers` and `id` — for complex tables

For multi-level/complicated header structures where `scope` alone can't unambiguously describe relationships:
```html
<th id="jan" scope="col">January</th>
<td headers="jan">$5,000</td>
```
More verbose but necessary when a cell is governed by, e.g., both a row header and a two-level column header.

## `rowspan` / `colspan`

Merge cells across rows/columns:
```html
<tr>
  <td rowspan="2">Merged across 2 rows</td>
  <td>Row 1</td>
</tr>
<tr>
  <td>Row 2</td>
</tr>
```
Common for grouped headers/summary cells — but every merge increases structural complexity for `scope`/`headers` to describe correctly, which is exactly why `headers`/`id` exists as a fallback for these cases.

## `<colgroup>`/`<col>`

Applies styling to entire columns without touching every cell:
```html
<table>
  <colgroup>
    <col style="background-color: #f0f0f0;">
    <col>
  </colgroup>
  <tr><td>A</td><td>B</td></tr>
</table>
```
Fairly niche today — CSS (`:nth-child`, Grid, `:has()`) often handles this more flexibly; still useful for things like fixed column widths on wide data tables.

## When to actually use `<table>`

Only for genuinely tabular data — real row/column relationships where headers describe the data (spreadsheets, comparison charts, schedules). **Never for page layout** — died with CSS's rise in the mid-2000s; using tables for layout today actively harms accessibility, since screen readers announce cell/row/column info that's meaningless for a layout table, creating confusing noise.

## Practical takeaway

`scope` on every `<th>` is non-negotiable for accessible tables — the single thing that separates "a table that looks fine visually" from "a table a screen reader user can actually use."

## Where we left off

Section 6 (Tables) covered in full: structure, `scope`, `headers`/`id` for complex cases, `rowspan`/`colspan`, `<colgroup>`/`<col>`, and the layout-vs-data-table distinction. No exercise done yet for this section — could revisit hands-on later (e.g. building a small accessible table with `scope`) if useful.

Next up per the HTML reference doc: **Section 7 — Forms (Full Surface)** — a large section covering `<form>` attributes, every `<input>` type, `<textarea>`/`<select>`/`<datalist>`, `<button>` types, `<label>`/`<fieldset>`/`<legend>`/`<output>`/`<progress>`/`<meter>`, validation attributes, the Constraint Validation API, FormData API, and autocomplete tokens.
