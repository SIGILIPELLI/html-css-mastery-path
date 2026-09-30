---
description: "Style components by the space they're given: container-type, @container rules, cqi units, style queries, and the zero-width trap."
---

# 06 · Container Queries

Media queries ask "how wide is the **viewport**?" But a component rarely cares about
the viewport. A product card might sit in a wide main column on one page and a narrow
sidebar on another — on the same screen. With media queries it can only adapt to the
window, so you end up writing variants (`.card--compact`) and remembering where to use
each. **Container queries** let the card ask "how wide is the space **I've been given**?"
and style itself accordingly, wherever it's placed.

## Two steps: declare a container, then query it

```css
/* 1. The wrapper becomes a query container */
.card-slot {
  container-type: inline-size;
}

/* 2. Rules inside @container apply based on that wrapper's size */
.card { display: grid; gap: 1rem; }

@container (width >= 30rem) {
  .card { grid-template-columns: 10rem 1fr; }   /* image beside text when there's room */
}
```

An `@container` rule queries the **nearest ancestor** that is a container. It can't
query the element itself — an element's styles can't depend on its own size, or changing
the style could change the size and loop forever. That's why you need the wrapper.

We placed the same card markup in three places on a 1280px page and read each card's
`grid-template-columns`:

```html
<div class="card-slot" style="width:300px"><article class="card">…</article></div>
<div class="card-slot" style="width:800px"><article class="card">…</article></div>
<div style="width:800px"><article class="card">…</article></div>  <!-- not a container -->
```

```text
300px container:  "300px"          one column
800px container:  "160px 624px"    image column + text
no container:     "800px"          one column — the query never matched
```

Same component, same CSS, two layouts, decided by the slot. The third card has no
container ancestor at all, so the `@container` rule simply doesn't apply — the base
styles are the fallback.

## `container-type` values

| Value | Queries you can make | Containment applied |
|-------|---------------------|---------------------|
| `inline-size` | width (inline size) | inline-size: its width can't depend on its children |
| `size` | width and height | size: neither dimension can depend on its children |
| `normal` (default) | only style queries | none |

Almost always use `inline-size`. `size` requires the container to have a height from
somewhere other than its content — we set `container-type: size` on a plain `div`
containing a paragraph, and it measured **0px** tall: size containment means "lay me out
as if I had no content."

## The zero-width trap

Containment has a second consequence that catches everyone once. We put an
`inline-size` container in two places — as a normal block, and as an item in a flex row
with no width set:

```text
block container width:      1264px
flex-item container width:  0px
```

A block takes the full available width, so it's fine. But a flex item (or an inline-block,
float, absolutely positioned element, or grid item in an `auto` track) normally sizes
itself **from its content** — and inline-size containment says its content can't
influence its width. With nothing else to go on, it collapses to zero.

The fix is to give such containers a size from outside: `flex: 1`, `width: 100%`, a grid
track like `1fr`, or a `min-width`.

## Named containers

With nested containers, a query matches the nearest one. Name them to target a specific
ancestor:

```css
.layout-main    { container: main / inline-size; }     /* shorthand: name / type */
.layout-sidebar { container: sidebar / inline-size; }

@container sidebar (width < 20rem) {
  .card .meta { display: none; }
}
```

## Container query units

Inside a container, lengths can be relative to it:

| Unit | 1% of the container's… |
|------|-------------------------|
| `cqi` | inline size (width in horizontal writing) |
| `cqb` | block size |
| `cqw`, `cqh` | width, height |
| `cqmin`, `cqmax` | smaller / larger dimension |

They make fluid typography work per component:

```css
.card h3 { font-size: clamp(1rem, 4cqi, 1.5rem); }
```

In our three placements the headings computed to:

```text
300px container: 16px   (4% of 300 = 12px, clamped up to 1rem)
800px container: 24px   (4% of 800 = 32px, clamped down to 1.5rem)
no container:    24px   (4% of the 1280px viewport = 51.2px, clamped to 1.5rem)
```

With no container ancestor, `cq*` units fall back to the equivalent **small viewport**
units, which is why the third heading behaved like a viewport-sized one.

## Style queries

`@container style(...)` queries a container's **computed style** — currently, in practice,
the value of a custom property:

```css
.card-slot { --variant: compact; }

@container style(--variant: compact) {
  .card .description { display: none; }
}
```

Both engines we tested (Chromium 153 and WebKit 26.6) applied a `style(--variant:
compact)` rule. Style queries let a parent pass a "mode" to descendants without adding
classes to each of them. Note that any element can be a style container (the default
`container-type: normal` supports style queries), and support in other browsers arrived
later than size queries — check current support before relying on them for essential
layout.

## Worked example: one card, three contexts

```html
<main class="main">
  <div class="card-slot"><article class="card">…</article></div>
</main>
<aside class="sidebar">
  <div class="card-slot"><article class="card">…</article></div>
</aside>
```

```css
.card-slot { container: card / inline-size; }

.card {
  display: grid;
  gap: 0.75rem;
  padding: 1rem;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
}

.card img { aspect-ratio: 16 / 9; object-fit: cover; width: 100%; }
.card h3 { font-size: clamp(1rem, 0.8rem + 2cqi, 1.5rem); margin: 0; }

/* medium: image beside text */
@container card (width >= 28rem) {
  .card { grid-template-columns: 12rem 1fr; align-items: start; }
  .card img { aspect-ratio: 1; }
}

/* large: more room for supporting information */
@container card (width >= 45rem) {
  .card { grid-template-columns: 16rem 1fr auto; }
  .card .price { align-self: center; font-size: 1.5rem; }
}
```

Drop the card into the main column, the sidebar, a modal, or a three-up grid, and it
picks the right layout for each — the page layout and the component layout are now
independent. Media queries still have their place: page-level layout (how many columns
the page has) is a viewport decision; component layout is a container decision.

## How It Actually Works

Container queries seem to create a cycle: styles depend on the container's size, and
size depends on styles. **Containment** breaks it. `container-type: inline-size` applies
inline-size containment, which guarantees the container's width is computed without
looking at its descendants. So the browser can:

1. Lay out the page up to the container and determine the container's width from its
   parent and its own styles alone.
2. Evaluate `@container` conditions against that width.
3. Recompute styles for the container's descendants and lay them out.

Nothing inside can change the container's width, so there's no loop — which is also
precisely why the container can't size to its contents (the zero-width trap) and why a
`size` container can't size to its content height.

Style and layout become interleaved here: container-query evaluation happens *during*
layout, so browsers have to be able to pause layout, restyle a subtree and resume.
That's a large part of why the feature took so long to arrive, and why containment is
mandatory rather than optional.

## Common mistakes

- **Trying to query the element itself** — you query an ancestor.
- **Forgetting `container-type`** on the wrapper, so nothing matches.
- **Containers that collapse to zero** inside flex rows, grids with `auto` tracks,
  or absolutely positioned elements.
- **`container-type: size`** on something that needs its content height.
- **Querying the wrong container** in nested layouts — name them.
- **Moving everything to container queries.** Page-level decisions still belong to media
  queries.

## Exercise

1. Turn the related-product card from your Level 2 project into a container-query
   component with two layouts. Place it in the main content and in a narrow sidebar.
2. Reproduce the zero-width trap: make a container a flex item, see it collapse, fix it.
3. Replace the card heading's size with a `cqi`-based `clamp()`. Compare its size in each
   placement using devtools.
4. Add a style query that hides the description when a parent sets `--variant: compact`.
5. In devtools, find the `container` badge next to container elements; click it to see
   which elements are queried by which rules.
