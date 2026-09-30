---
description: "Advanced CSS Grid: auto-fit vs auto-fill with minmax, the implicit grid, dense packing, subgrid for aligned cards, and grid alignment."
---

# 06 · Grid in Depth: auto-fit, Implicit Tracks & Subgrid

Lesson 05 defined grids with a fixed number of columns. The features in this lesson let a
grid decide for itself — how many columns fit, how tall new rows should be, how to fill
gaps — and let nested elements share their parent's tracks. Together they remove most of
the media queries you'd otherwise write for galleries and card layouts.

## The responsive gallery in one line

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}
```

Read it as: "make as many columns as fit, each at least 200px wide; then stretch them
equally to fill the row." No media queries: at 300px wide there's one column, at 700px
three, at 1200px five.

### `auto-fit` vs `auto-fill`

Both keywords create as many tracks as fit. They differ in what happens to tracks with no
items in them. We gave each version **only two items** and measured the computed columns
(`grid-template-columns` as reported by `getComputedStyle`) and item widths:

```text
container 300px
  auto-fit : tracks "300px"                       items [300, 300]   (two rows)
  auto-fill: tracks "300px"                       items [300, 300]
container 700px
  auto-fit : tracks "342px 342px 0px"             items [342, 342]
  auto-fill: tracks "222.656px 222.672px 222.672px" items [222.7, 222.7]
container 1200px
  auto-fit : tracks "592px 592px 0px 0px 0px"     items [592, 592]
  auto-fill: tracks "227.188px 227.203px ..." (5 tracks) items [227.2, 227.2]
```

Both created three tracks at 700px and five at 1200px. **`auto-fit` collapses the empty
tracks to 0px**, so the two items stretch across the whole row. **`auto-fill` keeps the
empty tracks**, so the items stay at their column width with blank space after them.

- Use `auto-fit` when items should fill the row (few items should look deliberate).
- Use `auto-fill` when items should keep a consistent size (a product grid where a
  half-empty last page shouldn't make cards huge).

### Preventing overflow on very small screens

`minmax(200px, 1fr)` has a hard 200px minimum. In a container narrower than 200px — a
phone with padding, or a narrow sidebar — the single column overflows. Cap the minimum at
the container width:

```css
.gallery {
  grid-template-columns: repeat(auto-fit, minmax(min(200px, 100%), 1fr));
}
```

`min(200px, 100%)` is 200px normally and 100% of the container when that's smaller.

## The implicit grid

When items are placed outside the tracks you defined — or you only defined columns — the
grid creates **implicit** tracks. Size them with `grid-auto-rows` / `grid-auto-columns`:

```css
.feed {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: minmax(150px, auto);   /* every new row at least 150px, taller if needed */
}
```

`grid-auto-flow` controls the direction auto-placement fills: `row` (default — fill each
row, add rows) or `column` (fill each column, add columns). `grid-auto-flow: column` with
`grid-auto-columns: 80%` and `overflow-x: auto` is a quick horizontal carousel.

## Dense packing

Auto-placement never goes *backwards* by default — once it has moved past a gap, the gap
stays empty. We placed four items in three columns: two that span two columns, then two
normal ones:

```css
.grid { grid-template-columns: repeat(3, 1fr); grid-auto-rows: 50px; }
.wide { grid-column: span 2; }
```

```text
sparse (default):     A: col 1, row 1 | B: col 1, row 2 | C: col 3, row 2 | D: col 1, row 3
grid-auto-flow: dense A: col 1, row 1 | B: col 1, row 2 | C: col 3, row 1 | D: col 3, row 2
```

In the default mode, B (two wide) didn't fit in row 1's remaining single column, so it
went to row 2 — leaving a hole at column 3, row 1 that nothing ever filled. The grid was
150px tall (three rows). With `dense`, C back-filled that hole and D fitted beside B; the
grid was 100px tall (two rows).

The catch: `dense` puts C **visually before** B even though it comes after in the source.
For a photo wall where order doesn't matter, fine. For anything the user reads or tabs
through in order, don't.

## Subgrid: aligning across nested elements

The classic card problem: a row of cards, each with a title, body and button. Titles and
bodies vary in length, so the buttons (and body text) start at different heights in each
card. Each card is its own box and can't know about its neighbours.

**Subgrid** lets a grid item adopt its parent's tracks for its own children:

```html
<div class="cards">
  <article class="card">
    <h3>Short</h3>
    <p>One line.</p>
    <a href="#">Buy</a>
  </article>
  <article class="card">
    <h3>A much longer title that wraps over lines</h3>
    <p>Body</p>
    <a href="#">Buy</a>
  </article>
  <article class="card">
    <h3>Mid title</h3>
    <p>A longer body that wraps onto several lines in the card.</p>
    <a href="#">Buy</a>
  </article>
</div>
```

```css
.cards {
  display: grid;
  grid-template-columns: repeat(3, 200px);
  gap: 16px;
}

.card {
  display: grid;
  grid-row: span 3;               /* each card occupies three of the parent's rows */
  grid-template-rows: subgrid;    /* ...and uses those rows for its children */
  gap: 8px;
}
```

We measured the top edge of each card's paragraph and button, first with the cards as
flex columns, then with subgrid:

```text
flex columns:  button tops [56, 78, 74]
subgrid:       p tops [52, 52, 52]   button tops [96, 96, 96]
```

With subgrid, all three cards share the same three row tracks: the title row is as tall
as the longest title *in any card*, and so on. Every paragraph and every button lines up.
Subgrid works in current versions of all major browsers (it was the last to arrive, in
Chromium 117, in 2023); in older browsers the cards fall back to normal grid rows.

Subgrid also inherits the parent's **gap** unless you override it (we did, with `8px`),
and line names from the parent grid are available inside the subgrid.

## Alignment in depth

```css
.grid {
  justify-items: start;         /* each item hugs the start of its cell horizontally */
  align-items: center;          /* and is centred vertically */
}
.grid > .logo { justify-self: end; }

.board {
  grid-template-columns: repeat(3, 100px);
  justify-content: center;      /* tracks are narrower than the container: centre them */
}
```

`*-items`/`*-self` align items **within their cells**; `*-content` aligns **the tracks
within the container**, which only matters when the tracks don't fill it.

## Worked example: a magazine-style layout with named lines

Named lines give you a full-bleed layout where some content breaks out of the text
column:

```css
.article {
  display: grid;
  grid-template-columns:
    [full-start] minmax(1rem, 1fr)
    [content-start] minmax(0, 65ch) [content-end]
    minmax(1rem, 1fr) [full-end];
}

.article > *        { grid-column: content; }   /* default: the text column */
.article > .bleed   { grid-column: full; }      /* images and quotes break out edge to edge */
```

Naming lines `content-start` and `content-end` creates an implicit area called `content`,
so `grid-column: content` means "from content-start to content-end." The outer
`minmax(1rem, 1fr)` columns act as gutters on small screens and as the margins that
centre the text on large ones. Every direct child of the article lands in the text column
unless you opt it into `full`.

## How It Actually Works

`repeat(auto-fit | auto-fill, ...)` is resolved **before** item placement: the browser
computes how many repetitions fit the container's size (using the track's minimum —
200px in our gallery — plus gaps), creates that many tracks, then places items. Only
afterwards, for `auto-fit`, does it collapse tracks that ended up empty to zero width
(and merge the gaps around them), after which `1fr` shares the reclaimed space.

The auto-placement algorithm maintains a **cursor**. In sparse mode the cursor only moves
forward: each item is placed at the first position at or after the cursor where it fits.
In dense mode, the search starts from the beginning of the grid for every item, so earlier
holes get filled.

Subgrid changes track sizing: the subgridded axis has no tracks of its own. Its children
participate in the **parent's** track sizing algorithm as if they were items spanning the
parent's tracks — which is exactly why a long title in one card can make the title row
taller for all of them.

## Common mistakes

- **`auto-fill` when you wanted items to stretch** (or vice versa).
- **`minmax(250px, 1fr)` overflowing narrow containers** — use `min(250px, 100%)`.
- **`dense` on content with a reading order.**
- **Setting `grid-template-rows: subgrid` without making the item span rows** in the
  parent — it then has one row to share.
- **Forgetting `grid-auto-rows`**, so implicit rows are sized to content when you wanted
  a consistent height.

## Exercise

1. Build a recipe gallery with `repeat(auto-fit, minmax(min(15rem, 100%), 1fr))`. Test with
   2 cards and 11 cards, then switch to `auto-fill` and compare.
2. Build the three-card example with flex columns, confirm the misaligned buttons, then
   convert to subgrid.
3. Make a photo wall where every fifth photo spans two columns. Compare sparse and dense
   packing.
4. Convert your recipe article to the named-lines layout and make the hero image
   `.bleed`.
