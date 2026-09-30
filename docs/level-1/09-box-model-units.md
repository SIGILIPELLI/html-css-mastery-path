---
description: "The CSS box model: content, padding, border and margin, box-sizing, margin collapsing, block vs inline, and when to use px, em, rem and ch."
---

# 09 · The Box Model, Display & Units

Layout in CSS starts from one fact: **every element generates a rectangular box**. The
box model decides how big that box is; the `display` type decides how it sits next to its
neighbours; units decide what "big" means. Flexbox and Grid in Level 2 are built on top
of these three ideas, so time spent here pays off for the rest of the course.

## The four areas of a box

```text
┌──────────────────────── margin ────────────────────────┐
│  ┌───────────────────── border ─────────────────────┐  │
│  │  ┌────────────────── padding ─────────────────┐  │  │
│  │  │                                            │  │  │
│  │  │              content (width × height)      │  │  │
│  │  │                                            │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

- **Content** — where text and children go.
- **Padding** — space inside the border. Takes the element's background.
- **Border** — drawn around the padding.
- **Margin** — space outside the border, always transparent. Pushes neighbours away.

The devtools Computed pane draws exactly this diagram for the selected element, with the
real numbers. Hovering an element in the Elements panel shades each area on the page.

### Shorthands

```css
.card {
  padding: 1rem;                 /* all four sides */
  padding: 1rem 2rem;            /* top/bottom  left/right */
  padding: 1rem 2rem 0.5rem;     /* top  left/right  bottom */
  padding: 1rem 2rem 0.5rem 0;   /* top right bottom left — clockwise */
  margin-inline: auto;           /* left and right (in left-to-right text) */
  padding-block: 1rem;           /* top and bottom */
  border: 1px solid #ddd;        /* width style colour */
  border-radius: 0.5rem;
}
```

`margin-inline` and `padding-block` are **logical** properties: "inline" follows the
direction text runs, "block" the direction paragraphs stack. In English they're
left/right and top/bottom. Level 4 · 05 explains why they matter for other languages.

## `box-sizing`: what does `width` mean?

By default (`box-sizing: content-box`), `width` sets the **content** width only; padding
and border are added on top. We gave two boxes the same declarations — `width: 300px;
padding: 20px; border: 5px solid;` — and changed only `box-sizing`. Chromium reported
their `offsetWidth` (border-box width):

```text
content-box: 350
border-box:  300
```

With `content-box`, the box is 300 + 2×20 + 2×5 = **350px** wide — not what anybody means
by "300px wide." With `border-box`, padding and border are carved out of the 300px.
Nearly every project starts with:

```css title="styles.css"
*, *::before, *::after {
  box-sizing: border-box;
}
```

## Display types: block, inline and friends

`display` controls two things: how the box behaves **towards its siblings** (outer
display) and how it lays out **its children** (inner display).

| `display` | Outer behaviour | Respects `width`/`height`? | Vertical margins? |
|-----------|-----------------|---------------------------|-------------------|
| `block` | Starts on a new line, fills available width | Yes | Yes |
| `inline` | Flows within a line of text | **No** | **No** (padding draws but doesn't push lines apart) |
| `inline-block` | Flows within text, but is a box inside | Yes | Yes |
| `none` | Not rendered at all, removed from the accessibility tree | — | — |
| `flex`, `grid` | Block outside, flex/grid layout for children (Level 2) | Yes | Yes |
| `inline-flex`, `inline-grid` | Inline outside, flex/grid inside | Yes | Yes |

`div`, `p`, `h1`, `ul`, `section` are block by default; `span`, `a`, `em`, `strong`,
`img`, `input` are inline. (Images and inputs are *replaced* elements, so they do respect
width and height.) Those defaults come from the user-agent stylesheet and have nothing
to do with what the element means — you can make an `<a>` `display: block` without
changing its semantics.

We gave an inline `<span>` `width: 200px; height: 50px; margin: 20px; padding: 10px` and
measured its box:

```text
getBoundingClientRect: 56 × 38
getComputedStyle(span).width: "200px"
```

The rendered box ignored `width` and `height` completely: 56px is the text plus
horizontal padding, 38px the line's font height plus vertical padding. Note that
`getComputedStyle` still reported `"200px"` — for inline boxes it returns the declared
value, even though layout doesn't use it. Measure boxes with
`getBoundingClientRect()`, not by reading `width`.

## Margin collapsing

Vertical margins between blocks **collapse**: two touching margins become one margin
equal to the larger, not their sum.

```css
.a { margin-bottom: 30px; }
.b { margin-top: 20px; }
```

```text
gap between A and B: 30
```

Not 50. This is deliberate — it lets you give every paragraph a top and bottom margin
without doubling the gaps between them.

The surprising case is **parent–child** collapsing. A child's top margin can collapse
through its parent if nothing separates them (no border, padding or inline content above
it), so the margin appears *outside* the parent:

```css
.parent  { background: #eee; }
.child   { margin-top: 40px; }
.parent2 { background: #eee; display: flow-root; }
```

We measured the gap above each parent and the child's offset inside it:

```text
.parent : 40px above the parent, child at 0px inside it
.parent2: 0px above the parent, child at 40px inside it
```

In the first case the grey background starts *below* the gap, as if the margin belonged
to the parent. `display: flow-root` creates a new **block formatting context**, which
contains its children's margins. Padding or a border on the parent also stops the
collapse. Margins never collapse inside flex and grid containers, which is one reason
layout inside those feels more predictable.

## Units

| Unit | Relative to | Good for |
|------|-------------|----------|
| `px` | A CSS pixel (1/96 inch at arm's length; not a device pixel) | Borders, shadows, small fixed details |
| `rem` | The **root** (`html`) element's font size | Font sizes, spacing — scales with user preferences |
| `em` | The **element's own** font size (for `font-size` itself: the parent's) | Padding that should scale with the text of a component |
| `%` | Usually the containing block's width | Fluid widths |
| `vw`, `vh` | 1% of viewport width / height | Full-screen sections, fluid type |
| `svh`, `lvh`, `dvh` | Small / large / dynamic viewport height | Mobile layouts where browser toolbars resize the viewport |
| `ch` | Width of the "0" glyph in the current font | Readable line lengths (`max-width: 65ch`) |
| `lh` | The element's computed line height | Spacing that matches lines of text |

Inside a `.card { font-size: 1.25em }` (20px, with the default 16px root), we measured:

```text
padding: 1em               -> 20px   (the card's font size)
padding: 1rem              -> 16px   (the root font size)
font-size: 1.25em, nested twice -> 31.25px  (20 × 1.25 × 1.25)
width: 60ch (monospace)    -> 468.062px
```

The nested line is the classic `em` trap: each level multiplies. A list inside a list
inside a `1.25em` card grows with every level. Use `rem` for font sizes and reserve `em`
for things that *should* scale with local text size — button padding, icon sizes.

### Why `rem` rather than `px` for text

Users can change their browser's default font size (Chrome: Settings → Appearance →
Font size). `rem` values scale with that setting; `px` font sizes ignore it. Setting
`html { font-size: 62.5% }` "to make rem maths easy" works but fights that setting in
subtle ways — prefer leaving the root at `100%` and thinking in rems.

## `overflow`

When content is bigger than its box:

```css
.snippet { max-height: 12rem; overflow: auto; }   /* scrollbars only when needed */
.clip    { overflow: hidden; }                    /* clipped, not scrollable */
.x-only  { overflow-x: auto; }                    /* the wide-table pattern, lesson 05 */
.nowrap  { overflow-wrap: anywhere; }             /* break long URLs instead of overflowing */
```

## How It Actually Works

Layout turns the DOM into a **box tree** and then assigns each box a position and size.
Normal flow — the layout you get without flex or grid — works in **formatting contexts**:

- In a **block formatting context (BFC)**, block boxes stack vertically, each taking the
  full available width minus its margins. Width flows **down** from the containing block;
  height flows **up** from the content. That asymmetry is why `width: 100%` usually works
  and `height: 100%` usually doesn't (the parent's height depends on its children).
- In an **inline formatting context**, inline boxes and text are laid out in **line
  boxes**. Each line's height comes from the tallest thing on it and `line-height`. Inline
  boxes don't have an independent height; their vertical padding and borders overlap
  neighbouring lines instead of pushing them away — you saw that in the span
  measurement.

Margin collapsing is a rule of block formatting contexts: adjoining vertical margins of
blocks **in the same BFC** combine. Anything that establishes a new BFC — `flow-root`,
`overflow` other than `visible`, floats, flex and grid items, absolutely positioned
elements — separates the margins inside it from the ones outside. `display: flow-root`
exists purely to create a BFC without side effects.

`auto` margins are how blocks are centred: with a fixed or max width, the remaining
horizontal space is split equally between `margin-left: auto` and `margin-right: auto`.
Vertical `auto` margins compute to 0 in normal flow — which is why `margin: auto` centres
horizontally but not vertically (in flex and grid it does both).

## Common mistakes

- **Forgetting `box-sizing: border-box`**, then fighting boxes that are wider than their
  `width`.
- **Setting width/height on inline elements** and wondering why nothing happens. Use
  `inline-block` or put it in a flex container.
- **Parent–child margin collapse** making a background start in the wrong place.
- **`height: 100%` with no parent height**. Consider `min-height: 100dvh` on the page
  wrapper, or let flex/grid stretch items.
- **Nested `em` font sizes** compounding.
- **`px` font sizes**, which ignore user font-size preferences.
- **`100vh` on mobile**, which can be taller than the visible area when toolbars are
  shown. `100dvh` (dynamic) or `100svh` (small) are usually what you want.

## Exercise

1. Add the `box-sizing` reset to your recipe stylesheet. Give a `.card` class a width,
   padding and border, and confirm in the Computed pane that the total width equals
   `width`.
2. Put an `h2` with a top margin as the first thing inside a coloured `section`. Observe
   the margin escape the section; fix it three ways (padding, border, `flow-root`).
3. Build a "tag" style (`<span class="tag">`) with padding and a border. Try to give it a
   height; then change it to `inline-block` and try again.
4. Convert every `px` font size on your page to `rem`. Change your browser's default font
   size and compare.
5. Limit your recipe's text column to `65ch` and centre it with `margin-inline: auto`.
