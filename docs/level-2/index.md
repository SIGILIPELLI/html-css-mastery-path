---
description: "Level 2 of HTML & CSS: typography, color and contrast, Flexbox and Grid in depth, positioning and stacking, responsive design and custom properties."
---

# Level 2 · Intermediate <span class="level-badge">Layout & Style</span>

Goal: lay out and style any page you're handed — a product page, a dashboard shell, an
article with a sidebar — so that it works from a 320px phone to a wide monitor, in light
and dark mode, without fighting the browser.

Level 1 gave you the model: elements generate boxes, the cascade assigns values, normal
flow places blocks and lines. Level 2 adds the tools that replace normal flow when you
need them, and the systems that make styles scale:

1. **Pick the layout mode per container.** Flexbox distributes space along one axis and
   lets content drive sizes; Grid defines tracks in two dimensions and fits items into
   them. Most real layouts nest both.
2. **Let things flow before you add breakpoints.** `clamp()`, `min()`, `auto-fit` grids and
   wrapping flex rows adapt continuously; media queries are for genuine changes of layout.
3. **Name your values.** Custom properties turn a stylesheet of hard-coded colours and
   sizes into a small, themeable system.

## Modules

1. [Typography & Web Fonts](01-typography.md) — font stacks, `@font-face`, `font-display`, unitless line-height, fluid `clamp()` sizes
2. [Color, Backgrounds & Gradients](02-color-backgrounds.md) — `oklch`, `color-mix`, relative colours, contrast ratios, gradients
3. [Flexbox Fundamentals](03-flexbox-fundamentals.md) — axes, alignment, `gap`, wrapping, everyday patterns
4. [Flexbox in Depth](04-flexbox-in-depth.md) — grow and shrink arithmetic, `flex: 1` vs `auto`, the `min-width: auto` trap
5. [CSS Grid Fundamentals](05-grid-fundamentals.md) — tracks, `fr`, line placement, named areas
6. [Grid in Depth](06-grid-in-depth.md) — `auto-fit` vs `auto-fill`, implicit grid, dense packing, subgrid
7. [Positioning & Stacking Contexts](07-positioning-stacking.md) — absolute, fixed, sticky, and why `z-index: 9999` loses
8. [Responsive Design & Media Queries](08-responsive-design.md) — mobile-first, range syntax, dark mode, reduced motion
9. [Custom Properties & Theming](09-custom-properties.md) — inheritance, fallbacks, the invalid-value trap, `@property`
10. [Project — A Responsive Product Page](10-project-product-page.md) — every technique in one tested page

## Before you start

You should be comfortable with everything in [Level 1](../level-1/index.md), especially
the cascade (08) and the box model (09) — Flexbox and Grid sizing build directly on them.

Keep devtools open throughout. Chrome, Edge, Firefox and Safari all have **Flexbox and
Grid overlays** (click the `flex` or `grid` badge next to an element in the Elements
panel) that draw lines, gaps and track sizes on the page. Most of this level's "why is it
that size?" questions are answered by turning an overlay on.

## How the examples were checked

As in Level 1, every block of `text` output is something we measured in a real browser —
widths from `getBoundingClientRect()`, values from `getComputedStyle()`, feature support
from `CSS.supports()` — using headless Chromium 153 and, where noted, WebKit 26.6. Where
an explanation involves arithmetic (flex grow and shrink, `fr` tracks), the lesson shows
the arithmetic *and* the measured result, so you can check one against the other.

!!! note "About browser support"
    Everything taught as a default technique in this level — Flexbox, Grid including
    subgrid, `gap`, `clamp()`, custom properties, `@property`, `oklch()`, `color-mix()`,
    range media queries and `light-dark()` — works in current versions of Chrome, Edge,
    Firefox and Safari. Features that were newer or less evenly supported when this course
    was written are called out where they appear, and Level 4 · 08 covers how to use new
    CSS safely with `@supports`.
