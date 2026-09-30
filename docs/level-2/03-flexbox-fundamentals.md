---
description: "Lay out rows and columns with Flexbox: main and cross axes, justify-content, align-items, gap, wrapping, ordering, and measured positions for each option."
---

# 03 · Flexbox Fundamentals

Flexbox is a layout mode for arranging items **in one direction** — a row or a column —
and distributing space among them. It's the right tool for navigation bars, button
groups, a media object (image beside text), a card's internal layout, centring
something, or pushing a footer to the bottom. For two-dimensional layouts — rows *and*
columns lining up — use Grid (lessons 05–06).

## Turning it on

```html
<nav class="toolbar">
  <a href="/">Home</a>
  <a href="/recipes">Recipes</a>
  <a href="/about">About</a>
</nav>
```

```css
.toolbar { display: flex; }
```

The element with `display: flex` is the **flex container**; its direct children are
**flex items**. (Grandchildren are not affected — each level decides its own layout.)
Immediately:

- Items sit side by side in a row, in source order.
- Each item is only as wide as its content.
- Items stretch to the height of the tallest item.

## The two axes

Every flex container has a **main axis** (the direction items are placed) and a **cross
axis** (perpendicular to it). `flex-direction` sets the main axis:

| `flex-direction` | Main axis | Cross axis |
|------------------|-----------|------------|
| `row` (default) | horizontal, in the text direction | vertical |
| `column` | vertical, top to bottom | horizontal |
| `row-reverse`, `column-reverse` | reversed | — |

Every alignment property is defined relative to these axes, not to "horizontal" and
"vertical" — which is why `justify-content: center` centres horizontally in a row and
vertically in a column.

## Main axis: `justify-content`

We placed three 100px items in a 600px flex container and measured each item's left
offset in Chromium:

```text
flex-start     [0, 100, 200]
center         [150, 250, 350]
space-between  [0, 250, 500]     first and last touch the edges, equal gaps between
space-around   [50, 250, 450]    equal space around each item (so edges get half)
space-evenly   [75, 250, 425]    equal space everywhere, including the edges
```

`justify-content` only has an effect when there is free space left over. If items have
`flex-grow` (next lesson), they absorb the space first and there's nothing to justify.

## Cross axis: `align-items` and `align-self`

| `align-items` | Effect |
|---------------|--------|
| `stretch` (default) | Items fill the container's cross size (if they have no fixed height) |
| `flex-start` / `start` | Items at the top (in a row) |
| `center` | Items centred |
| `flex-end` / `end` | Items at the bottom |
| `baseline` | Items aligned by their first line of text |

We put two paragraphs — one line and two lines — in a 100px-tall row with the default
`stretch`. Both measured **100px** tall. That default is why equal-height columns became
trivial with Flexbox. An individual item can opt out with `align-self`.

`baseline` is the best choice when items have different font sizes — a big price next to
a small "per month" label reads naturally only when their text sits on the same line.

## `gap`

```css
.toolbar { display: flex; gap: 1rem; }
```

`gap` puts space **between** items only — not before the first or after the last. It's
far better than margins on items, which you'd then have to remove from the last one. You
can set row and column gaps separately: `gap: 2rem 1rem` (row gap, then column gap).

## Wrapping

By default, flex items stay on one line and shrink to fit. `flex-wrap: wrap` lets them
move onto new lines:

```css
.tags { display: flex; flex-wrap: wrap; gap: 0.5rem; }
```

Four 150px items with a 20px gap in a 500px container: we measured the tops of each item
as `[0, 0, 0, 40]`. Three fit on the first line (150 × 3 + 20 × 2 = 490px ≤ 500px), the
fourth wrapped to a second line 40px down (its 20px height plus the 20px row gap).

When items wrap, each line is its own flex line: items on line two don't line up in
columns with line one. If you need them to, you want Grid.

## Common patterns

### Centre anything

```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100dvh;
}
```

(Or with Grid: `display: grid; place-items: center;`.)

### Push one item to the end

```css
.site-header { display: flex; align-items: center; gap: 1rem; }
.site-header .account { margin-left: auto; }
```

An `auto` margin in a flex container absorbs **all** the free space on that side. This
is how you split a toolbar into "left group" and "right group" without extra wrappers.
(Use `margin-inline-start: auto` if the page may be right-to-left.)

### Media object

```html
<article class="media">
  <img src="avatar.jpg" alt="" width="64" height="64">
  <div class="media-body">
    <h3>Priya Rao</h3>
    <p>Made the soup twice this week. Roasting the garlic is the key.</p>
  </div>
</article>
```

```css
.media { display: flex; gap: 1rem; align-items: flex-start; }
.media img { flex-shrink: 0; border-radius: 50%; }
```

`flex-shrink: 0` stops the image from being squashed when the text is long.

### Sticky footer

```css
body {
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
}
main { flex: 1; }   /* main grows to fill, pushing the footer down */
```

### Card with button at the bottom

```css
.card { display: flex; flex-direction: column; }
.card .actions { margin-top: auto; }
```

In a row of cards with different amounts of text, each card stretches to the row height
(default `stretch` in the parent), and `margin-top: auto` pins the buttons to the bottom
of every card.

## `order` and reversing — and why to be careful

```css
.featured { order: -1; }   /* default is 0; lower comes first visually */
```

`order` and the `-reverse` directions change the **visual** order only. The DOM order —
which is the tab order and the screen-reader reading order — stays the same. If a sighted
keyboard user tabs through links that jump around the screen, that's a WCAG failure
("focus order" and "meaningful sequence"). Use visual reordering only for things that
aren't interactive or where the order genuinely doesn't matter; otherwise change the
HTML.

## How It Actually Works

When a box becomes a flex container, it establishes a **flex formatting context** for its
children. Layout proceeds roughly like this:

1. **Collect items.** Each in-flow child becomes a flex item; runs of text directly in
   the container are wrapped in anonymous items. Absolutely positioned children are
   taken out.
2. **Determine each item's hypothetical main size** from its `flex-basis` (next lesson).
3. **Collect items into lines.** With `nowrap`, one line. With `wrap`, fill each line
   until the next item wouldn't fit.
4. **Resolve flexible lengths** on each line: grow or shrink items to use the free
   space (next lesson).
5. **Cross size.** Work out each line's cross size (the tallest item, in a row); stretch
   items with `align-self: stretch` to it.
6. **Main-axis alignment:** distribute any remaining free space with `auto` margins
   first, then `justify-content`.
7. **Cross-axis alignment:** `align-items`/`align-self` within each line, and
   `align-content` to distribute multiple lines within the container.

Step 6's order is why `margin-left: auto` beats `justify-content` — auto margins eat the
free space before `justify-content` ever sees it.

Also note what *doesn't* happen in a flex container: margins don't collapse, `float`
has no effect on items, and `vertical-align` is ignored.

## Common mistakes

- **Using Flexbox for a grid of cards** and fighting the last row. Use Grid.
- **Expecting grandchildren to be flex items.**
- **Margins for spacing between items** instead of `gap`.
- **`justify-content` with growing items** — there's no free space to distribute.
- **Visual reordering of links and buttons** that breaks keyboard order.
- **Forgetting `flex-shrink: 0`** on images and icons, which then squash.
- **`align-items: center` when you meant `justify-content: center`** — check which axis
  is which for the current `flex-direction`.

## Exercise

1. Make your recipe page's header a flex row: site name on the left, nav on the right
   using `margin-left: auto`, vertically centred.
2. Turn the "facts" list (serves/prep/cook) into a wrapping flex row with `gap`.
3. Build three cards with different text lengths in a flex row. Make them equal height,
   with buttons aligned at the bottom.
4. Put a comment "media object" under the recipe and test it with very long text — then
   remove `flex-shrink: 0` and watch the avatar.
5. In devtools, click the "flex" badge next to a flex container in the Elements panel to
   show the flex overlay. Toggle `justify-content` values from the Styles pane's flexbox
   editor.
