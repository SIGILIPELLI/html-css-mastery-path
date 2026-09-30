---
description: "Build accessible data tables: caption, thead and tbody, th with scope, spanning cells, complex headers, basic table styling and making wide tables scroll."
---

# 05 · Tables

Tables have a bad reputation because for about a decade they were used to lay out entire
websites. That use is long dead — CSS Grid and Flexbox do layout. But for their actual
purpose, **tabular data** where each value belongs to both a row and a column, nothing
beats a real `<table>`: it's navigable cell by cell with a screen reader, sortable,
copy-pasteable into spreadsheets, and printable.

The test is simple: if the content would make sense in a spreadsheet, with meaningful
row and column headers, it's a table. If you're only trying to put things side by side,
it isn't.

## A complete, accessible table

```html title="hours.html"
<table>
  <caption>Opening hours</caption>
  <thead>
    <tr>
      <th scope="col">Day</th>
      <th scope="col">Open</th>
      <th scope="col">Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Monday</th>
      <td>09:00</td>
      <td>17:00</td>
    </tr>
    <tr>
      <th scope="row">Saturday</th>
      <td>10:00</td>
      <td>14:00</td>
    </tr>
  </tbody>
</table>
```

| Element | Role |
|---------|------|
| `<table>` | The table itself |
| `<caption>` | Its title. Must be the first child. Becomes the table's accessible name. |
| `<thead>`, `<tbody>`, `<tfoot>` | Row groups: header rows, body rows, footer (e.g. totals) rows |
| `<tr>` | A row |
| `<th>` | A header cell |
| `<td>` | A data cell |
| `scope="col"` / `scope="row"` | Which cells this header applies to |

Here is the accessibility tree Chromium built from that markup, as reported by
Playwright:

```text
- table "Opening hours":
  - caption: Opening hours
  - rowgroup:
    - row "Day Open Close":
      - columnheader "Day"
      - columnheader "Open"
      - columnheader "Close"
  - rowgroup:
    - row "Monday 09:00 17:00":
      - rowheader "Monday"
      - cell "09:00"
      - cell "17:00"
    - row "Saturday 10:00 14:00":
      - rowheader "Saturday"
      - cell "10:00"
      - cell "14:00"
```

Because "Monday" is a `rowheader` and "Close" a `columnheader`, a screen reader landing
on `17:00` can announce "Monday, Close, 17:00". Without `<th>`s it would announce
"17:00" and leave the user to count columns.

## Spanning cells

`colspan` and `rowspan` merge cells:

```html
<table>
  <caption>Class schedule</caption>
  <thead>
    <tr>
      <th scope="col">Time</th>
      <th scope="col">Monday</th>
      <th scope="col">Tuesday</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">09:00</th>
      <td rowspan="2">Knife skills (2 h)</td>
      <td>Bread basics</td>
    </tr>
    <tr>
      <th scope="row">10:00</th>
      <!-- Monday cell is covered by the rowspan above -->
      <td>Bread shaping</td>
    </tr>
    <tr>
      <th scope="row">11:00</th>
      <td colspan="2">Lunch — all groups</td>
    </tr>
  </tbody>
</table>
```

The easy mistake is forgetting that a `rowspan` cell *occupies* the slot in later rows,
so those rows need one fewer cell. Count slots, not tags.

## Grouped headers

When a header covers a group of columns, use `scope="colgroup"` (and `<colgroup>` to
define the group):

```html
<table>
  <caption>Nutrition per serving</caption>
  <colgroup><col></colgroup>
  <colgroup span="2"></colgroup>
  <colgroup span="2"></colgroup>
  <thead>
    <tr>
      <td rowspan="2"></td>
      <th scope="colgroup" colspan="2">Small bowl</th>
      <th scope="colgroup" colspan="2">Large bowl</th>
    </tr>
    <tr>
      <th scope="col">kcal</th><th scope="col">Protein</th>
      <th scope="col">kcal</th><th scope="col">Protein</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Tomato soup</th><td>150</td><td>4 g</td><td>240</td><td>6 g</td></tr>
  </tbody>
</table>
```

Once a table has more than two levels of headers, or headers in the middle of the body,
consider splitting it into several simpler tables. For genuinely complex cases, each
`<td>` can list its headers explicitly with `headers="id1 id2"` pointing to the `id`s of
its `<th>`s — precise, but tedious to maintain.

## Styling tables

The browser's defaults are minimal. A clean, readable baseline:

```css title="styles.css"
table {
  border-collapse: collapse;   /* adjacent cell borders merge into one line */
  width: 100%;
  font-variant-numeric: tabular-nums;  /* digits all the same width, so columns align */
}

caption {
  text-align: start;
  font-weight: 600;
  padding-block: 0.5rem;
}

th, td {
  padding: 0.5rem 0.75rem;
  border-bottom: 1px solid #ddd;
  text-align: start;
  vertical-align: top;
}

thead th {
  border-bottom-width: 2px;
}

td.num, th.num {
  text-align: end;   /* right-align numbers so place values line up */
}

tbody tr:nth-child(even) {
  background: #f7f7f7;  /* zebra striping */
}
```

`border-collapse: collapse` is almost always what you want; the default `separate` model
draws a border around every cell with gaps between them.

### Wide tables on small screens

A table with many columns will be wider than a phone. Don't squash it; let it scroll
horizontally inside a container:

```html
<section class="table-scroll" aria-labelledby="nutrition-caption" tabindex="0">
  <table>
    <caption id="nutrition-caption">Nutrition per serving</caption>
    …
  </table>
</section>
```

```css
.table-scroll {
  overflow-x: auto;
}
```

The `tabindex="0"` lets keyboard users focus the region and scroll it with the arrow
keys. A `<section>` with an accessible name (here from `aria-labelledby`) is exposed as a
**region** landmark, so screen-reader users hear what they've focused. You'll also see
`<div role="region">` used for this; html-validate flags that with "Prefer to use the
native `<section>` element", which is how we ended up with the version above. (Level 3
explains `tabindex` and landmarks.)

## Worked example: the DOM always has a `tbody`

As lesson 01 showed, if you write rows directly inside `<table>`, the parser inserts a
`<tbody>`. That has a direct CSS consequence:

```css
table > tr { background: yellow; }        /* matches nothing */
table > tbody > tr { background: yellow; } /* works */
table tr { background: yellow; }           /* works (descendant) */
```

The table DOM API reflects the same structure. On our "Opening hours" table, Chromium
reported:

```text
t.rows.length                          -> 3
t.tBodies[0].rows[1].cells[2].textContent -> "14:00"
t.caption.textContent                  -> "Opening hours"
```

`rows` includes the header row, which is why there are 3.

## How It Actually Works

Tables have their own layout algorithm, separate from normal block layout. By default
(`table-layout: auto`) the browser must look at the content of **every cell** to work out
column widths: it computes each column's minimum width (the longest unbreakable word or
image) and preferred width (content on one line), then distributes the available width.
That's why a long URL in one cell can widen the whole column, and why very large tables
can be slow to lay out — nothing can be placed until everything has been measured.

`table-layout: fixed` changes the rule: column widths come from the table's width and the
first row's cells (or `<col>` elements) only. Layout is faster and predictable, and long
content wraps or overflows instead of stretching columns:

```css
table.fixed { table-layout: fixed; width: 100%; }
table.fixed td { overflow-wrap: anywhere; }
```

On the accessibility side, the browser maps `<th scope="col">` to `columnheader` and
`scope="row"` to `rowheader`. Without `scope`, browsers use heuristics to guess whether a
`<th>` heads a row or a column — usually right for simple tables and often wrong for
complex ones. Writing `scope` removes the guessing.

## Common mistakes

- **Tables for layout.** Screen readers announce "table, 3 columns, 2 rows" for your
  page header.
- **Data in `<div>`s styled to look like a table.** It loses all the header
  relationships. If you need a grid of `<div>`s to look like a table *and* you can't use a
  table, that's an ARIA `grid`/`table` role job — much more work (Level 3 · 04).
- **No `<caption>`** — the table has no name in the accessibility tree.
- **All cells as `<td>`** with bold styling for headers.
- **Wrong cell counts** when using `rowspan`/`colspan`, creating ragged rows.
- **Squeezing wide tables** into phone widths with tiny text instead of allowing scroll.

## Exercise

1. Build a table of your weekly schedule or a comparison of three products with at least
   four attributes. Use `caption`, `thead`, `tbody`, and `th` with correct `scope` for
   both rows and columns.
2. Add a `<tfoot>` row with totals or a summary.
3. Style it with the CSS above. Add a `.num` class to numeric columns and check that the
   digits line up.
4. Make it wider than 375px, wrap it in the scrolling region, and test it in the device
   toolbar using only the keyboard: Tab to the region and scroll with arrow keys.
5. In devtools, select a data cell and find its computed accessible name/role in the
   Accessibility pane.
