---
description: "Style, layout, paint and composite explained, layout thrashing measured, containment, content-visibility, and profiling rendering."
---

# 01 · The Rendering Pipeline & CSS Performance

Most pages are slow because of what they load, not how they render (lesson 02 covers
loading). But once a page is interactive — scrolling, animating, updating in response to
input — rendering cost decides whether it feels smooth. At 60 frames per second the
browser has about **16.7ms per frame** for everything: running your JavaScript,
recalculating styles, layout, painting and compositing. This lesson is about knowing
which of those stages your CSS triggers, and measuring instead of guessing.

## The pipeline, revisited

Level 1 · 01 sketched the pipeline for the first render. The same stages run again on
every update:

```mermaid
flowchart LR
  JS[JavaScript / event] --> S[Style recalc]
  S --> L[Layout]
  L --> P[Paint]
  P --> C[Composite]
```

- **Style** — find which rules match which elements and compute values. Cost grows with
  the number of elements whose styles are invalidated.
- **Layout** (also called *reflow*) — compute geometry. A change to one element's size
  can move everything after it, and its ancestors.
- **Paint** — record drawing commands for text, colours, borders, shadows, images, into
  layers.
- **Composite** — combine layers on the GPU, applying transforms and opacity.

Any change starts the pipeline at the stage it affects and runs everything after it:

| You change… | Stages that run |
|-------------|----------------|
| `width`, `height`, `margin`, `padding`, `top`, `font-size`, text content, adding/removing elements | style → **layout** → paint → composite |
| `color`, `background`, `box-shadow`, `border-radius`, `outline` | style → **paint** → composite |
| `transform`, `opacity` (on an element with its own layer) | style → **composite** only |

That table is why Level 3 · 08 recommended animating `transform` and `opacity`.

## Layout thrashing — measured

Layout normally happens **once per frame**, lazily: you can change a hundred styles, and
the browser lays out once before painting. But some JavaScript APIs need up-to-date
geometry *right now* — `offsetWidth`, `offsetHeight`, `getBoundingClientRect()`,
`scrollTop`, `getComputedStyle()` for layout properties, and others. Reading one after a
style change **forces a synchronous layout**. Alternating writes and reads forces one
per iteration.

We measured two ways of widening 1,000 elements by one pixel each, in Chromium, counting
layouts with the DevTools protocol's `Performance.getMetrics`:

```js
// interleaved: read, write, read, write…
for (const item of items) {
  const w = item.offsetWidth;          // read (forces layout if anything changed)
  item.style.width = (w + 1) + 'px';   // write (invalidates layout)
}

// batched: all reads, then all writes
const widths = items.map((item) => item.offsetWidth);
items.forEach((item, i) => { item.style.width = (widths[i] + 1) + 'px'; });
```

```text
interleaved: 146.7 ms, layouts=1000
batched:       2.4 ms, layouts=2
interleaved: 145.8 ms, layouts=1001
batched:       2.0 ms, layouts=2
```

(Two runs of each. Absolute times depend on the machine — these were on the laptop this
course was written on — but the *ratio* is the point.) The interleaved loop forced a
thousand layouts and took roughly 60–70 times longer — more than eight frames' worth of
work for a trivial change. Batching reads before writes, or deferring writes to
`requestAnimationFrame`, turns it back into one layout.

## What makes style and layout expensive

- **Size of the DOM.** Style and layout scale with the number of elements affected. Pages
  with tens of thousands of nodes are slow to update however clever the CSS.
- **Invalidation scope.** Changing a class on `<body>` can require restyling every
  element; changing one on a leaf element restyles only it and maybe its siblings. Toggle
  classes as close as possible to the elements that change.
- **Layout scope.** A size change inside a normal-flow document can affect the whole page.
  Flex and grid containers with content-sized tracks may need to measure all their items.
- **Expensive paints.** Large `box-shadow` and `filter: blur()` areas, and big areas
  repainted every frame (for example a gradient on a `position: fixed` background
  during scrolling).

Selector matching itself is rarely the problem on normal pages (Level 1 · 07 explains
why); don't rewrite readable selectors for speed without a measurement.

## Containment

The `contain` property is a promise to the browser that part of the page is independent,
so work can be skipped or limited:

| Value | Promise | Effect |
|-------|---------|--------|
| `layout` | Nothing inside affects layout outside, and vice versa | Layout changes inside don't re-layout the rest of the page |
| `paint` | Nothing inside paints outside the box | Clips to the box; skipped entirely when off-screen |
| `size` | The box's size doesn't depend on its content | Size can be computed without laying out children |
| `inline-size` | Same, for width only | Used by container queries (Level 3 · 06) |
| `style` | Counters and quotes don't escape | Rarely needed on its own |
| `content` | `layout paint style` | A safe default for independent widgets |
| `strict` | `size layout paint style` | Needs an explicit size |

```css
.comment, .feed-item, .widget { contain: content; }
```

## `content-visibility: auto`

The most powerful rendering optimisation in CSS: skip style, layout and paint for
off-screen sections entirely, until they approach the viewport.

```css
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 1500px;  /* placeholder size until it's been rendered once */
}
```

We built a long page — 400 sections, each with 20 paragraphs and a 20-row table — and
measured the initial rendering work in Chromium:

```text
content-visibility: visible  LayoutDuration=122ms  RecalcStyleDuration=28ms  scrollHeight=475933
content-visibility: auto     LayoutDuration=5ms    RecalcStyleDuration=3ms   scrollHeight=599396
```

Layout work fell from 122ms to 5ms, because only the sections near the viewport were laid
out. Notice the **scrollHeight** changed too: off-screen sections were sized using the
`contain-intrinsic-size` guess (1500px) rather than their real height (about 1190px).
That's the trade-off — a wrong estimate makes the scrollbar jump as sections render. The
`auto` keyword in `contain-intrinsic-size: auto 1500px` tells the browser to remember
each section's real size once it has been rendered, so the estimate only matters until
then.

Use it for long, sectioned content (articles, feeds, documentation pages). Don't use it on
content near the top of the page, which would be rendered immediately anyway.
Content skipped this way is still in the DOM and the accessibility tree, and find-in-page
still works.

## `will-change`

```css
.drawer { will-change: transform; }
```

This hints that a property will animate, so the browser can promote the element to its
own compositor layer in advance. It costs GPU memory for every layer, so apply it only to
elements that are about to animate (add it on hover or just before the animation, remove
it after), never to large numbers of elements "to make them fast."

## How to profile

1. **Chrome/Edge devtools → Performance panel.** Record while you perform the slow
   interaction. The flame chart shows each frame's Scripting (yellow), Rendering — style
   and layout (purple) — and Painting (green). Purple blocks labelled "Layout" with a red
   triangle and "Forced reflow" warnings point straight at thrashing, with a link to the
   line of JavaScript that caused it.
2. **Rendering tab** (devtools → More tools → Rendering): "Paint flashing" highlights
   repainted areas in green; "Layout shift regions" shows shifting content; "Frame
   rendering stats" shows a live FPS meter.
3. **Layers panel**: see which elements have their own compositor layers and why.
4. **Performance insights / Lighthouse** for loading-related issues (lesson 02).

Throttle the CPU (Performance panel → CPU: 4× slowdown) — your development laptop is much
faster than a typical phone.

## How It Actually Works

Browsers keep **dirty bits**: when a style change arrives, the affected element is marked
as needing style recalc; if a layout-affecting value changed, it and its ancestors are
marked as needing layout. Nothing is recomputed yet. At the next frame (or when
JavaScript forces it by reading geometry), the engine walks only the dirty parts of the
tree. That's what "invalidation" means, and it's why batching works: a hundred writes set
a hundred dirty bits, and one layout clears them all.

Containment works by cutting the propagation of those dirty bits. With `contain: layout`,
a dirty descendant doesn't mark ancestors outside the boundary; with `content-visibility:
auto`, off-screen subtrees are treated as if `display` were skipped for layout and paint
until an intersection observer inside the engine notices them nearing the viewport.

Compositing is the reason `transform` and `opacity` are special. Painted layers are
stored as textures on the GPU. Moving or fading a whole layer only changes how the
compositor draws those textures — the main thread isn't involved, so the animation keeps
running even while JavaScript is busy.

## Common mistakes

- **Reading layout properties inside loops that write styles.**
- **Animating layout properties** (`top`, `width`, `height`) instead of transforms.
- **Sprinkling `will-change`** across a page.
- **Huge DOMs** — rendering thousands of rows instead of paginating or virtualising.
- **`content-visibility: auto` without `contain-intrinsic-size`**, which makes skipped
  sections zero-height and the scrollbar wildly inaccurate.
- **Optimising selectors** instead of measuring.

## Exercise

1. Reproduce the thrashing experiment in your own browser with `performance.now()`, then
   record it in the Performance panel and find the "Forced reflow" warnings.
2. Turn on Paint flashing and hover over, scroll and interact with your Level 3 gallery.
   What repaints? Is anything repainting that shouldn't?
3. Add `content-visibility: auto` to the sections of a long page (a long article or
   documentation page). Compare the Performance panel's "Rendering" time before and after,
   and watch the scrollbar as you scroll.
4. Find one animation on a site you use that animates a layout property, and rewrite it
   with transforms.
