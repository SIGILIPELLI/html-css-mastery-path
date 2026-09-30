---
description: "How Flexbox sizes items: flex-basis, grow and shrink maths checked against measured widths, flex: 1 vs auto, and the min-width: auto trap."
---

# 04 · Flexbox in Depth: How Sizes Are Resolved

Lesson 03 was about *positioning* items. This lesson is about *sizing* them — the part
of Flexbox people learn by trial and error. The algorithm is short and entirely
predictable. Once you've done the arithmetic a few times, you'll know exactly what
`flex: 1` does, why one column refuses to shrink, and why a code block blows your layout
out of its container.

## The three properties

Each flex item has:

- **`flex-basis`** — its starting size along the main axis, before free space is shared
  out. Default `auto` (use the item's `width`/`height`, or its content size if that's
  `auto` too).
- **`flex-grow`** — how many shares of **positive** free space it takes. Default `0`
  (don't grow).
- **`flex-shrink`** — how readily it gives up space when there's **negative** free space
  (overflow). Default `1`.

Write them with the `flex` shorthand:

| Shorthand | Means | Use for |
|-----------|-------|---------|
| `flex: initial` (the default) | `0 1 auto` | Content-sized, can shrink, won't grow |
| `flex: auto` | `1 1 auto` | Start at content size, share extra space |
| `flex: 1` | `1 1 0%` | **Ignore content size**, share all space equally |
| `flex: none` | `0 0 auto` | Rigid: content size, never grows or shrinks |
| `flex: 0 0 250px` | fixed 250px | Sidebars, fixed columns |
| `flex: 2 1 200px` | start at 200px, grow with weight 2 | |

Notice that `flex: 1` sets the basis to **0**, not `auto`. That single detail explains most
confusion about it.

## Growing: sharing positive free space

The algorithm: **free space = container size − sum of bases** (minus gaps). Each growing
item gets `free space × (its grow ÷ total grow)` added to its basis.

We put two items in a 600px container, `flex: 1 1 100px` and `flex: 2 1 200px`, and
measured them in Chromium:

```text
widths: [200, 400]
```

The arithmetic: bases sum to 300, so free space is 300. Grow factors total 3; item one
gets 1/3 × 300 = 100 → **200px**; item two gets 2/3 × 300 = 200 → **400px**. Note that
item two isn't "twice as wide" — it gets twice the *extra space*.

### `flex: 1` vs `flex: auto`

Three items with different amounts of text, in a 600px container:

```text
flex: 1     -> [200, 200, 200]
flex: auto  -> [120.47, 323.95, 155.58]
```

With `flex: 1`, every basis is 0, so *all* 600px is free space, split equally — the items
end up equal regardless of content. With `flex: auto`, each starts at its content width
and only the leftover space is shared equally, so items with more text stay wider.
Choose `flex: 1` for equal columns, `flex: auto` for "content-sized, but fill the row".

## Shrinking: sharing negative free space

When the bases add up to more than the container, items shrink. Shrinking is **weighted
by basis**: an item's share of the overflow is proportional to `flex-shrink × flex-basis`.
Otherwise a small item could be shrunk to nothing while a big one barely changes.

Three items `flex: 0 1 400px`, `0 1 200px`, `0 1 200px` in 600px:

```text
widths: [300, 150, 150]
```

Overflow is 800 − 600 = 200. Scaled shrink factors are 400, 200, 200 (total 800). The
first item loses 400/800 × 200 = 100 → 300px; each of the others loses 50 → 150px.
Everything shrank by the same **proportion** (25%).

## The `min-width: auto` trap

Here's the most common Flexbox bug. A 300px container with a fixed 100px sidebar and a
`flex: 1` main column containing a long line of code in a `<pre>`:

```html
<div class="layout">
  <div class="side"></div>
  <div class="main">
    <pre>const url = "https://example.com/a/very/long/path/that/does/not/wrap";</pre>
  </div>
</div>
```

```css
.layout { display: flex; width: 300px; }
.side   { flex: 0 0 100px; }
.main   { flex: 1; }
```

You'd expect `.main` to be 200px. We measured it:

```text
.main width (default):                 546
.main width (min-width: 0; overflow: auto): 200
```

The main column grew to **546px**, overflowing the 300px container. The reason: flex
items have `min-width: auto` by default, which for flex items means "don't shrink below
my **content's minimum size**." The content's minimum size is the unbreakable
`<pre>` line, so the item refuses to be narrower than that line, whatever `flex-basis`
says.

The fix is to allow the item to be smaller than its content:

```css
.main {
  flex: 1;
  min-width: 0;      /* allow shrinking below content size */
}
.main pre { overflow-x: auto; }   /* and let the long content scroll */
```

The same thing happens with long URLs, wide tables, images with large intrinsic sizes and
nested flex rows. In a column layout it's `min-height: auto` that bites. Whenever a flex
child overflows mysteriously, try `min-width: 0` (or `overflow: hidden`, which also
removes the automatic minimum) on it first.

## Worked example: a layout that behaves at every width

A sidebar layout that puts the sidebar *beside* the content when there's room and
*above* it when there isn't — with no media query:

```html
<div class="with-sidebar">
  <aside class="sidebar">…filters…</aside>
  <main class="content">…results…</main>
</div>
```

```css
.with-sidebar {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
}

.sidebar {
  flex: 1 1 15rem;        /* ideally 15rem wide, may grow */
}

.content {
  flex: 999 1 0;          /* grows far faster than the sidebar */
  min-width: 60%;         /* but when it can't be at least 60% wide, wrap */
}
```

How it works: `.content` has a basis of 0 and a huge grow factor, so it takes nearly all
the free space and the sidebar stays close to 15rem. When the container gets narrow
enough that `.content` can't meet its `min-width: 60%` on the same line as the sidebar,
flex-wrap puts it on its own line, where both items grow to full width. We measured it
at three container widths:

```text
1000px: .sidebar w=241 top=0   .content w=735 top=0
 700px: .sidebar w=240 top=0   .content w=436 top=0
 400px: .sidebar w=400 top=0   .content w=400 top=42
```

At 400px, a sidebar of 240px plus the 24px gap would leave `.content` less than its
240px minimum (60% of 400), so it wrapped below and both went full width. This "sidebar"
pattern (popularised by Heydon Pickering and Andy Bell's *Every Layout*) responds to the
**container's** width, not the viewport's — useful when the same component appears in a
wide main area and a narrow column.

## How It Actually Works

The spec's "resolve flexible lengths" step is iterative:

1. Compute each item's **hypothetical main size**: its flex-basis, clamped by its
   min/max sizes (with `min-width: auto` resolved to the content-based minimum).
2. Sum them (plus gaps). If the sum is less than the container, use grow factors; if
   more, use shrink factors.
3. Distribute the free space in proportion to the factors (for shrink, `flex-shrink ×
   basis`).
4. **Clamp**: if any item's new size violates its `min-width` or `max-width`, freeze it at
   that limit.
5. Recalculate free space with the frozen items removed and repeat from step 3 for the
   rest, until nothing changes.

That loop explains the `min-width: auto` result: in the first pass the main item was
offered a width near 200px, violated its content-based minimum, got frozen at 546px, and
the layout simply overflowed.

It also explains why `max-width` on a growing item doesn't "waste" space — the item is
frozen at its max and the space is redistributed to its siblings in the next pass.

A final subtlety: `flex-basis: auto` means "look at `width`" (or `height` in a column),
and only if that is also `auto` does it use the **content size**. So `width: 200px;
flex-basis: auto` gives a 200px basis, while `flex-basis: 0` ignores `width` entirely for
the basis — but `width`'s sibling `min-width` still applies as a clamp.

## Common mistakes

- **Thinking `flex: 1` means "grow"** and being surprised that content size is ignored.
- **Expecting `flex-grow: 2` to make an item twice as wide.** It gets twice the *extra
  space*.
- **Overflowing flex items from long content** — add `min-width: 0`.
- **`width` on flex items expecting it to be final.** It's only the basis (when
  `flex-basis` is `auto`), and items can still grow or shrink from it.
- **Fixed sidebars without `flex-shrink: 0`**, which shrink on small screens.

## Exercise

1. Predict, then measure in devtools: three items with `flex: 1 1 100px`, `flex: 1 1 200px`
   and `flex: 2 1 0` in a 700px container.
2. Predict the widths of `flex: 0 1 300px` and `flex: 0 3 300px` in a 400px container.
   (Hint: scaled shrink factors are 300 and 900.)
3. Recreate the `min-width: auto` trap with a long URL instead of a `<pre>`, then fix it.
4. Build the worked-example sidebar layout. Resize the browser and find the width at which
   it wraps. Put the same component inside a 400px-wide box and confirm it wraps there too.
