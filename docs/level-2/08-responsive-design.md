---
description: "Responsive design that holds up: mobile-first media queries, range syntax, content-based breakpoints, dark mode and reduced motion."
---

# 08 · Responsive Design & Media Queries

A responsive page adapts to whatever it's shown on — a 320px phone, a tablet in either
orientation, a 2560px monitor, a browser zoomed to 200%, a phone in dark mode with
reduced motion switched on. The foundations are already in place from earlier lessons:
the viewport meta tag (Level 1 · 02), fluid images (Level 1 · 04), relative units and
`clamp()` (lesson 01), wrapping flex rows (lesson 03) and `auto-fit` grids (lesson 06).
Media queries are the last piece: switching layout or behaviour when conditions change.

## Fluid first, queries second

Before reaching for a breakpoint, ask whether the layout could just *flow*:

```css
.container { width: min(100% - 2rem, 70rem); margin-inline: auto; }   /* gutters + max width */
.cards     { display: grid; grid-template-columns: repeat(auto-fit, minmax(min(16rem, 100%), 1fr)); gap: 1rem; }
h1         { font-size: clamp(2rem, 1.2rem + 3vw, 3.25rem); }
.stack > * + * { margin-block-start: 1.5rem; }
```

Every one of those adapts continuously with no breakpoint. Media queries are best kept
for genuine *changes of layout* — navigation that collapses, a sidebar that moves.

## Media query syntax

```css
@media (width >= 48rem) {
  .page { grid-template-columns: 16rem 1fr; }
}

@media (40rem <= width < 64rem) { … }        /* a range */
@media (orientation: landscape) and (height < 30rem) { … }
@media print { … }                            /* Level 4 · 07 */
@media not (hover: hover) { … }
```

The **range syntax** (`width >= 48rem`) is supported by all current browsers and reads
far better than the older `min-width` / `max-width` form. They mean the same thing:
`(min-width: 48rem)` ≡ `(width >= 48rem)`. The one subtle difference is at the boundary:
`max-width: 48rem` includes 48rem exactly, so pairing `max-width: 48rem` with
`min-width: 48rem` gives an overlap at exactly 48rem; with range syntax you write
`width < 48rem` and `width >= 48rem` and the overlap disappears.

## Mobile first

Write the base styles for the narrowest screen, then add complexity as space allows with
`>=` queries:

```css
.grid { display: grid; gap: 1rem; }                       /* one column: phones */

@media (width >= 40rem) {
  .grid { grid-template-columns: 1fr 1fr; }                /* two columns */
}

@media (width >= 64rem) {
  .grid { grid-template-columns: repeat(3, 1fr); }         /* three columns */
}
```

We rendered that at three viewport widths in Chromium and read the computed
`grid-template-columns`:

```text
375px  -> 375px
800px  -> 392px 392px
1280px -> 416px 416px 416px
```

Mobile-first tends to produce less CSS: the narrow layout is usually the simpler one, so
you *add* rules as the screen grows instead of *undoing* desktop rules for phones.

### Where to put breakpoints

Not at device widths. Devices change every year; your content doesn't. Start with the
narrow layout, widen the browser slowly, and add a breakpoint **where the design starts
to look wrong** — lines too long, cards too stretched, a nav that has room to expand.
Most sites need two or three breakpoints.

### `em`/`rem` in media queries don't use your root font size

Relative units in media queries are based on the browser's **initial** font size (16px by
default, or whatever the user set) — not on the `font-size` you set on `html`. We set
`:root { font-size: 20px }` and loaded a page with `@media (min-width: 40em)` in a 700px
window. The query **matched**: 40em was treated as 640px (40 × 16), not 800px (40 × 20).
That's actually useful — `em`/`rem` breakpoints follow the *user's* font-size setting, so
a user with larger default text gets the narrower layout sooner, which is what they need.

## Responding to capabilities and preferences

Width isn't the only thing that varies.

### Input: `hover` and `pointer`

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { transform: translateY(-2px); }       /* mouse and trackpad users */
}

@media (pointer: coarse) {
  .toolbar button { min-block-size: 44px; }          /* touch: bigger targets */
}
```

Don't hide essential information behind `:hover` — on touch screens there is no hover.

### Colour scheme: dark mode

```css
:root {
  color-scheme: light dark;       /* browser UI (form controls, scrollbars) follows too */
  --bg: #ffffff;
  --text: #1c1c1c;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #141414;
    --text: #ececec;
  }
}

body { background: var(--bg); color: var(--text); }

/* or, per property, with light-dark() */
.card { background: light-dark(#fff, #1f1f1f); }
```

We loaded this in Chromium with the colour scheme emulated as light, then as dark:

```text
light: body background rgb(255, 255, 255), light-dark() text rgb(0, 0, 0),       matchMedia dark: false
dark:  body background rgb(20, 20, 20),    light-dark() text rgb(255, 255, 255), matchMedia dark: true
```

`light-dark()` only works when `color-scheme` includes both values. Custom properties
(lesson 09) make dark mode a matter of redefining a handful of variables.

### Reduced motion

Some people get dizzy or nauseous from motion on screen; operating systems have a setting
to reduce it, and CSS can read it:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

This blanket rule is a safety net. Better still is to write motion as an opt-in —
`@media (prefers-reduced-motion: no-preference) { ... }` — so animations only exist for
people who haven't asked to avoid them. Level 3 · 08 goes deeper.

### Other preference queries

- `prefers-contrast: more` — the user wants higher contrast.
- `forced-colors: active` — Windows High Contrast mode is on; the browser replaces your
  colours with the user's palette. Borders and outlines survive, backgrounds and shadows
  don't — so never rely on a background colour alone to show a button's edge.

## Responsive images and viewport units recap

- `srcset` / `sizes` choose image **files** (Level 1 · 04). `<picture>` with `media`
  changes **crops**.
- `100vw` includes the vertical scrollbar on desktop, so `width: 100vw` causes horizontal
  scroll on pages with a scrollbar. Use `100%` for widths.
- `100vh` on mobile is the height with the browser toolbars *hidden*, so it's taller than
  the visible area when they're shown. Use `100dvh` (dynamic), `100svh` (smallest) or
  `100lvh` (largest).

## Worked example: navigation that collapses without JavaScript

```html
<header class="site-header">
  <a class="logo" href="/">Weeknight Kitchen</a>
  <details class="menu">
    <summary>Menu</summary>
    <nav aria-label="Main">
      <ul>
        <li><a href="/recipes">Recipes</a></li>
        <li><a href="/techniques">Techniques</a></li>
        <li><a href="/about">About</a></li>
      </ul>
    </nav>
  </details>
</header>
```

```css
.site-header { display: flex; flex-wrap: wrap; align-items: center; gap: 1rem; }
.menu { margin-inline-start: auto; }
.menu ul { list-style: none; margin: 0; padding: 0; }

@media (width >= 48rem) {
  .menu > summary { display: none; }                 /* no toggle on wide screens */
  .menu ul { display: flex; gap: 1.5rem; }
}
```

`<details>` gives you an accessible disclosure widget (Level 3 · 09). On wide screens we
hide the summary with CSS — but a closed `<details>` hides its content, so the menu must
also be open there. That needs one line of script (`menu.open = matchMedia('(width >= 48rem)').matches`,
re-run on change), or use the `open` attribute by default and let small screens close it.
It's a good illustration of a real limitation: CSS can't change an element's *state*,
only its presentation.

## How It Actually Works

Media queries are evaluated against the **viewport** (for screen media) — specifically
the layout viewport whose width is set by the viewport meta tag. When a query's result
changes (the window is resized, the device rotates, the user switches to dark mode), the
browser invalidates styles for the affected rules, recomputes styles and re-runs layout.
Rules inside a non-matching `@media` block aren't discarded; they're simply skipped during
cascade until the condition changes.

JavaScript can read the same conditions with `window.matchMedia(query)`, which returns
an object with a `.matches` boolean and fires a `change` event — the right way to sync
script behaviour with CSS breakpoints instead of reading `innerWidth` on every resize.

Media queries only know about the **viewport**. A component in a narrow sidebar on a wide
screen still sees a wide viewport. That limitation is exactly what container queries
solve (Level 3 · 06).

## Common mistakes

- **Missing viewport meta tag** — no media query will ever fire on phones.
- **Breakpoints at device widths** instead of where the content breaks.
- **Desktop-first CSS** full of rules that undo other rules.
- **`width: 100vw`** causing horizontal scroll.
- **`max-width` / `min-width` pairs overlapping** at the boundary pixel.
- **Hover-only interactions.**
- **Dark mode that forgets form controls** — add `color-scheme`.
- **Assuming responsive only means width** — test zoom at 200%, dark mode, reduced motion
  and keyboard use too.

## Exercise

1. Make your recipe page's layout mobile-first with two breakpoints you choose by
   resizing slowly. Write down the widths where things started to look wrong.
2. Add dark mode with `prefers-color-scheme` and custom properties. Test it with Chrome
   devtools → Rendering → "Emulate CSS media feature prefers-color-scheme".
3. Add a reduced-motion rule and test it with the same Rendering panel.
4. Zoom your page to 200% at a 1280px window. Does it still work? (At 200% zoom, the
   layout viewport is effectively 640px, so your mobile layout should appear.)
5. Build the collapsing navigation and make the menu open on wide screens using a small
   `matchMedia` listener.
