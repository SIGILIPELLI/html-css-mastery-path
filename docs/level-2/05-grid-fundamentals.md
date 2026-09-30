---
description: "Build two-dimensional layouts with CSS Grid: tracks, the fr unit, repeat(), gap, placing items by line numbers and by named template areas."
---

# 05 · CSS Grid Fundamentals

Flexbox arranges items along one line. **Grid** arranges them in rows *and* columns at
the same time, so things line up in both directions. That makes it the natural tool for
page layouts (header, sidebar, content, footer), card galleries, forms with aligned labels,
dashboards and anything else you'd sketch on graph paper.

The mental model: you define **tracks** (columns and rows) on the container, which creates
**grid lines** between them and **cells** at their intersections. Then you place items
into cells — or let the browser place them automatically.

## Defining columns and rows

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 2fr;
  gap: 20px;
}
```

That's three columns: a fixed 200px track, then two flexible ones. The **`fr` unit** is a
fraction of the *leftover* space after fixed tracks and gaps are subtracted. We measured
three items in a 1000px container:

```text
widths: [200, 253.33, 506.67]
```

1000 − 200 (fixed) − 40 (two 20px gaps) = 760px left over, split 1:2 → 253.33 and
506.67. Like `flex-grow`, `fr` shares **free space**, not total space.

Rows work the same way with `grid-template-rows`. If you don't define rows, the grid
creates them as needed, sized to their content — the **implicit grid** (lesson 06).

### `repeat()` and `minmax()`

```css
.gallery { grid-template-columns: repeat(4, 1fr); }          /* 4 equal columns */
.form    { grid-template-columns: max-content 1fr; }          /* labels | fields */
.sidebar { grid-template-columns: minmax(12rem, 20%) 1fr; }  /* at least 12rem, ideally 20% */
.pattern { grid-template-columns: repeat(2, 1fr 3fr); }       /* 1fr 3fr 1fr 3fr */
```

`minmax(MIN, MAX)` gives a track a range. Content keywords are useful in track sizes:
`min-content` (as narrow as possible without overflow — the longest word), `max-content`
(as wide as the content wants on one line), `auto` (usually "fit the content, but stretch
if there's room"), and `fit-content(300px)` (content size, capped at 300px).

### The `1fr` minimum trap

`1fr` is actually shorthand for `minmax(auto, 1fr)` — and that `auto` minimum means a
track won't shrink below its content's minimum size. It's the grid version of the Flexbox
`min-width: auto` trap. Three `1fr` columns in an 800px grid, with one very long
unbreakable word in the middle:

```text
repeat(3, 1fr)            -> [211.72, 376.55, 211.73]
repeat(3, minmax(0, 1fr)) -> [266.66, 266.67, 266.66]
```

With plain `1fr`, the middle column grew to fit the word and the others shrank. With
`minmax(0, 1fr)`, the columns stay equal and the word overflows its cell (handle that with
`overflow-wrap: anywhere` on the content). When you need columns to be *truly* equal —
for example, a pricing table — use `minmax(0, 1fr)`.

## `gap`

```css
.grid { gap: 1.5rem; }          /* rows and columns */
.grid { gap: 2rem 1rem; }       /* row-gap column-gap */
```

Gaps are only between tracks, never at the outer edges.

## Placing items by line number

Grid lines are numbered from 1 at the start edge. With three columns there are four
column lines; line −1 is always the last line of the explicit grid.

```css
.header { grid-column: 1 / -1; }      /* from the first line to the last: full width */
.feature { grid-column: 2 / 4; grid-row: 1 / 3; }  /* two columns wide, two rows tall */
.wide { grid-column: span 2; }        /* two tracks wide, wherever auto-placement puts it */
```

`grid-column: A / B` is shorthand for `grid-column-start: A; grid-column-end: B`. Items can
overlap if you place them in the same cells — later items paint on top (control with
`z-index`), which is useful for layering a caption over an image without absolute
positioning.

## Named areas

For page layouts, drawing the layout as text is clearer than line numbers:

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100dvh;
}

.page > header { grid-area: header; }
.page > aside  { grid-area: sidebar; }
.page > main   { grid-area: main; }
.page > footer { grid-area: footer; }
```

Each string is a row; each word is a cell; repeating a name spans cells. Areas must be
rectangles. A `.` marks an empty cell.

We rendered this (with `min-height: 400px` instead of `100dvh`, in a 900px-wide grid)
and measured each element:

```text
header: x=0   y=0   w=900 h=18
aside:  x=0   y=18  w=200 h=364
main:   x=200 y=18  w=700 h=364
footer: x=0   y=382 w=900 h=18
```

In our HTML, `<main>` came *before* `<aside>` — but the aside still appeared on the left,
because placement comes from the areas, not the source order. That's powerful and, as with
Flexbox `order`, a potential accessibility problem: reading and tab order follow the
**source**, so keep the source in a sensible order.

The `auto 1fr auto` rows give the header and footer their content height and give the
middle row everything else — the sticky-footer layout in one declaration.

### Rearranging with media queries

Areas make responsive rearrangement a matter of redrawing the map:

```css
.page {
  grid-template-columns: 1fr;
  grid-template-areas:
    "header"
    "main"
    "sidebar"
    "footer";
}

@media (width >= 48rem) {
  .page {
    grid-template-columns: 200px 1fr;
    grid-template-areas:
      "header  header"
      "sidebar main"
      "footer  footer";
  }
}
```

## Alignment

Grid uses the same alignment vocabulary as Flexbox, in two dimensions:

| Property | Aligns | Within |
|----------|--------|--------|
| `justify-items` / `justify-self` | items horizontally (inline axis) | their cell |
| `align-items` / `align-self` | items vertically (block axis) | their cell |
| `justify-content` / `align-content` | the whole set of tracks | the container, when tracks are smaller than it |
| `place-items` | shorthand for `align-items justify-items` | |

Items default to `stretch`, filling their cell. `place-items: center` on a grid with one
cell is the shortest way to centre something in CSS.

## Worked example: a form with aligned labels

```html
<form class="settings">
  <label for="name">Display name</label>
  <input id="name" name="name" autocomplete="nickname">

  <label for="email">Email address</label>
  <input id="email" name="email" type="email" autocomplete="email">

  <label for="bio">Short bio</label>
  <textarea id="bio" name="bio" rows="3"></textarea>

  <button type="submit">Save</button>
</form>
```

```css
.settings {
  display: grid;
  grid-template-columns: max-content minmax(0, 1fr);
  gap: 0.75rem 1rem;
  align-items: baseline;
  max-width: 40rem;
}

.settings button { grid-column: 2; justify-self: start; }

@media (width < 30rem) {
  .settings { grid-template-columns: 1fr; }
  .settings button { grid-column: auto; }
}
```

No wrapper `div`s needed: labels auto-place in column 1, fields in column 2, and the label
column is exactly as wide as the longest label. On narrow screens it collapses to one
column, label above field.

## How It Actually Works

Grid layout runs in phases:

1. **Place items.** Items with explicit positions are placed first; then the
   **auto-placement algorithm** walks the remaining items in order, putting each into the
   next free area big enough for it, adding implicit rows as needed.
2. **Size the tracks** with the track sizing algorithm: initialise each track's base size
   and growth limit from its sizing function (fixed, content-based or flexible); resolve
   content-based sizes by looking at the items in those tracks (spanning items last);
   maximise tracks up to their growth limits if there's space; then expand `fr` tracks,
   dividing leftover space in proportion to their flex factors — while respecting each
   `fr` track's minimum (which is how the long word won in the example above).
3. **Lay out items** inside their grid areas and apply alignment.

Columns are sized before rows, because in horizontal writing, row heights depend on how
text wraps, which depends on column widths.

A key difference from Flexbox: in Grid, the **container** defines the tracks and items fit
into them; in Flexbox, **items** have sizes and the container distributes space among them.
That's the practical rule for choosing: layout-first (you know the columns) → Grid;
content-first (items decide, the container adapts) → Flexbox.

## Common mistakes

- **Using `1fr` where columns must stay equal** — use `minmax(0, 1fr)`.
- **Forgetting that `fr` shares leftover space**, not total space.
- **Line numbers off by one** — three columns have *four* lines.
- **Visual reordering with areas** that no longer matches the DOM order.
- **Non-rectangular area maps** (an L-shape); the whole `grid-template-areas` declaration
  is invalid and ignored.
- **Wrapping everything in extra `div`s** — grid only affects direct children, so plan
  the markup, but you often need fewer wrappers than with older techniques.

## Exercise

1. Build the named-area page layout for your recipe site with a header, a sidebar
   ("More recipes"), main content and footer. Make it single-column below 48rem.
2. Build the settings form from the worked example and test it at 320px and 1024px wide.
3. Make a 4-column "photo mosaic" where the first photo spans two columns and two rows.
4. Enable the grid overlay in devtools (click the `grid` badge) with line numbers and
   area names showing. Check your line numbers.
5. Reproduce the `1fr` long-word trap and fix it with `minmax(0, 1fr)`.
