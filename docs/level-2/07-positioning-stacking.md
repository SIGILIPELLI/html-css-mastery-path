---
description: "CSS positioning explained: relative, absolute, fixed, sticky, containing blocks, and why z-index: 9999 can still lose to a stacking context."
---

# 07 · Positioning & Stacking Contexts

Normal flow, Flexbox and Grid place boxes *relative to each other*. The `position`
property is for the exceptions: a badge in the corner of a card, a header that sticks
while you scroll, a dropdown that floats above the content, a tooltip. It's also the
source of the most frustrating CSS bug there is — the `z-index` that doesn't work no
matter how big you make it. That bug has a precise explanation, and it's in this lesson.

## The five values

| `position` | In normal flow? | Offsets (`top`/`left`/…) are relative to | Scrolls with page? |
|------------|-----------------|-------------------------------------------|--------------------|
| `static` (default) | yes | offsets ignored | yes |
| `relative` | yes (space kept) | its own normal position | yes |
| `absolute` | **no** (space collapses) | the **containing block**: nearest positioned ancestor | yes |
| `fixed` | **no** | the viewport (usually) | no |
| `sticky` | yes | its scroll container, once a threshold is crossed | until its parent scrolls away |

`inset: 0` is shorthand for `top: 0; right: 0; bottom: 0; left: 0`.

## `relative`

Moves the element visually without affecting anything around it — the space it would
have taken is kept. On its own, that's rarely useful. Its main job is to be an **anchor**:
it makes the element a containing block for absolutely positioned descendants, and it
lets `z-index` apply.

## `absolute` and the containing block

An absolutely positioned element is removed from the flow and positioned against its
**containing block** — the padding box of its nearest ancestor with `position` other than
`static` (or with a `transform`, `filter` or `contain` set). If there's none, it's the
initial containing block: the viewport-sized box at the top of the page.

We put a 20×20 badge with `position: absolute; top: 0; right: 0;` inside a card, twice —
once with the card left `static`, once with `position: relative` on the card — and
measured where each badge landed in a 1280px-wide page (the second card's box starts at
x=70, y=260 and is 400px wide):

```text
no positioned ancestor:     badge at (1260, 0)   <- top-right corner of the page
card has position: relative: badge at (450, 260)  <- top-right corner of the card
```

That's the whole pattern:

```css
.card { position: relative; }
.card .badge { position: absolute; top: 0.5rem; right: 0.5rem; }
```

Absolute elements shrink to fit their content unless you give them a width, or set both
`left` and `right` (which stretches them between the two).

## `fixed`

Positioned against the viewport, so it stays put while the page scrolls — cookie banners,
floating action buttons, full-screen overlays. Two cautions:

- Fixed elements cover content. A fixed header needs matching `padding-top` on the page,
  and in-page anchor links need `scroll-margin-top` on their targets so headings don't
  land underneath it:

    ```css
    :target, h2[id] { scroll-margin-top: 5rem; }
    ```

- If *any* ancestor has a `transform`, `filter`, `perspective`, `contain: paint` or
  `backdrop-filter`, that ancestor becomes the containing block and "fixed" stops being
  fixed to the viewport. A modal inside an animated container is the usual victim.

## `sticky`

A hybrid: the element stays in normal flow until the scroll position would move it past
its threshold (`top: 0`), then it sticks — but **only within its parent**. When the
parent scrolls away, the sticky element goes with it.

```css
.section-heading { position: sticky; top: 0; background: white; }
```

We put a sticky 50px header at the top of a 2000px-tall section in a 600px-tall window
and measured its top edge after scrolling:

```text
scrolled 1000px: header top = 0      <- stuck to the top of the viewport
scrolled 2500px: header top = -550   <- its section has scrolled past, header went with it
```

Sticky's two classic failures:

1. **It needs room to move.** If the sticky element's parent is only as tall as the
   element itself (for example, you made the `<header>` wrapper sticky instead of the
   whole page's first child), there's nowhere to stick.
2. **An ancestor with `overflow` other than `visible` becomes its scroll container.** We
   put the same header inside a `div` with `overflow: hidden` and scrolled the page 500px:
   the header's top was **−500** — it scrolled away as if it weren't sticky, because it
   was sticking to the `div` (which never scrolls), not to the page. Use `overflow: clip`
   if you only need clipping; it doesn't create a scroll container.

## `z-index` and stacking

When positioned elements overlap, `z-index` decides which paints on top — higher in
front. It applies to positioned elements (and to flex and grid items, even unpositioned).

That description is incomplete in a way that causes real bugs. Here's a test page with a
site header (`z-index: 10`) and a modal (`z-index: 9999`) that should cover it:

```html
<div class="header"></div>
<div class="modal-wrap">
  <div class="modal">modal</div>
</div>
```

```css
.header     { position: relative; z-index: 10; height: 300px; }
.modal-wrap { position: relative; opacity: 0.99; }
.modal      { position: fixed; inset: 100px; z-index: 9999; background: #fff; }
```

We asked Chromium what's on top at a point where both overlap
(`document.elementFromPoint(200, 150)`):

```text
with opacity: 0.99 on .modal-wrap -> header
without the opacity               -> modal
```

`z-index: 9999` **lost to `z-index: 10`**, and removing an unrelated-looking `opacity`
fixed it.

## How It Actually Works

Painting order isn't one global list sorted by `z-index`. It's a tree of **stacking
contexts**. A stacking context is a group of elements that are painted together, as a
unit, and whose `z-index` values only compete **with each other**.

The root element forms the first stacking context. A new one is created by, among others:

- `position: relative | absolute` **with** a `z-index` other than `auto`
- `position: fixed | sticky` (always)
- `opacity` less than 1
- `transform`, `filter`, `perspective`, `clip-path`, `mask`, `backdrop-filter`
- `isolation: isolate`
- `mix-blend-mode` other than `normal`
- `will-change` naming any of the above
- `contain: layout` or `contain: paint`
- flex and grid items with a `z-index` other than `auto`

Inside the test page, `opacity: 0.99` made `.modal-wrap` a stacking context with an
effective `z-index` of `auto` (treated as 0 for ordering). The modal's 9999 now only
ranks it *inside* `.modal-wrap`. At the root level, the comparison is `.header` (10)
against `.modal-wrap` (0) — and the header wins, taking everything inside `.modal-wrap`
down with it. No `z-index` on the modal can ever escape its parent's context.

Within a single stacking context, the paint order is:

1. The context's own background and borders
2. Descendants with negative `z-index`
3. In-flow, non-positioned blocks
4. Floats
5. Inline content
6. Positioned descendants with `z-index: auto` or `0`, in source order
7. Positioned descendants with positive `z-index`, lowest first

So the debugging procedure for "z-index doesn't work" is: walk up from each of the two
elements to find the stacking context each belongs to, find their ancestors that are
**siblings within the same context**, and compare those. Chromium's devtools has a
"Layers" panel, and Edge adds a "3D View" with a z-index mode, that visualise this.

Two practical defences:

- **Keep `z-index` values small and meaningful** — a handful of named layers rather than
  9999 races:

    ```css
    :root { --z-dropdown: 10; --z-sticky: 20; --z-overlay: 30; --z-modal: 40; --z-toast: 50; }
    ```

- **Use `isolation: isolate` on components** that use z-index internally. It creates a
  stacking context with no visual side effects, so a component's internal layering can't
  leak out and fight the rest of the page.

And the modern escape hatch: the native `<dialog>` element opened with `showModal()`, and
the `popover` attribute, render in the **top layer** — above every stacking context on
the page, whatever the z-index values. Level 3 · 09 covers both; for modals and popups,
they make this whole class of bug disappear.

## Common mistakes

- **Absolute positioning without a positioned ancestor**, so the element flies to the
  page corner.
- **`z-index` on a static element** (outside flex/grid) — it does nothing.
- **Escalating `z-index` to 9999** instead of finding the stacking context.
- **A transformed ancestor breaking `position: fixed`.**
- **Sticky inside `overflow: hidden`** — use `overflow: clip`.
- **Absolute positioning for layout** that Grid or Flexbox would handle; absolutely
  positioned elements don't affect siblings, so content overlaps as soon as text changes
  length.

## Exercise

1. Add a "New" badge to the top-right corner of a card. Remove `position: relative` from
   the card and watch where it goes.
2. Make your recipe page's section headings sticky. Then wrap the article in a `div` with
   `overflow: hidden` and see stickiness break; fix it with `overflow: clip`.
3. Recreate the modal/opacity test. Confirm the bug with `document.elementFromPoint`, then
   fix it two ways: remove the stacking context, and move the modal to be a direct child
   of `body`.
4. Define z-index custom properties for your site's layers and replace any raw numbers.
