---
description: "Level 2 project: a responsive, dark-mode-aware product page using Grid, Flexbox, subgrid, clamp() and custom properties, tested at three widths."
---

# 10 · Project — A Responsive Product Page

This project pulls every Level 2 technique into one page: an online shop's product page
for a cast-iron skillet. It has a photo gallery that rearranges itself, a buy box that
sticks while you scroll the photos, a specifications grid with no media queries, a row of
related-product cards aligned with subgrid, fluid type, and a full dark theme driven by
custom properties.

As in Level 1, the reference solution was built and checked, not just written: it's
validated, audited with axe-core in light and dark mode, and measured at three viewport
widths. (It's a course exercise — the shop, prices and links aren't real.)

## Requirements

1. **Mobile-first**: one column on phones; photos and the buy box side by side from
   48rem; a thumbnail rail beside the main photo from 64rem.
2. **Fluid**: headings sized with `clamp()`, spacing with a fluid custom property, a
   container with `min()` gutters.
3. **Specifications** as a `<dl>` in an `auto-fit` grid — no media queries.
4. **Related products** as cards in an `auto-fill` grid, with subgrid so titles,
   descriptions and prices line up across each row, and the whole card clickable.
5. **A sticky site header** and a sticky buy box on wide screens, with z-index managed by
   a custom property.
6. **Light and dark themes** from a single set of semantic custom properties, with
   `color-scheme` so form controls follow.
7. **Size options** as real radio buttons in a fieldset, visually styled as buttons, with
   the selected one highlighted.
8. **Motion** only for users who haven't asked for reduced motion, and hover effects only
   on devices that can hover.
9. No horizontal scrolling at 360px; html-validate and axe-core clean.

## Reference solution: the HTML

```html title="index.html"
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>28 cm Cast-Iron Skillet — Weeknight Kitchen Shop</title>
    <meta name="description" content="A pre-seasoned 28 cm cast-iron skillet for searing, roasting and baking. Sizes, specifications, care guide and related items.">
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <a class="skip-link" href="#main">Skip to product</a>

    <header class="site-header">
      <div class="container header-inner">
        <a class="logo" href="index.html">Weeknight Kitchen <span>Shop</span></a>
        <nav aria-label="Main">
          <ul class="nav-list">
            <li><a href="cookware.html" aria-current="page">Cookware</a></li>
            <li><a href="tools.html">Tools</a></li>
            <li><a href="basket.html">Basket (0)</a></li>
          </ul>
        </nav>
      </div>
    </header>

    <main id="main" class="container">
      <nav aria-label="Breadcrumb" class="breadcrumb">
        <ol>
          <li><a href="index.html">Shop</a></li>
          <li><a href="cookware.html">Cookware</a></li>
          <li><a href="skillet.html" aria-current="page">Cast-iron skillet</a></li>
        </ol>
      </nav>

      <div class="product">
        <section class="gallery" aria-label="Product photos">
          <img class="gallery-main" src="img/pan-front.png" width="1000" height="1000"
               alt="Black cast-iron skillet seen from above, with a short helper handle opposite the main handle">
          <img src="img/pan-side.png" width="1000" height="1000" loading="lazy"
               alt="The skillet from the side, showing its 5 cm walls and pouring lip">
          <img src="img/pan-handle.png" width="1000" height="1000" loading="lazy"
               alt="Close-up of the long handle with its hanging hole">
        </section>

        <div class="summary">
          <h1>Cast-Iron Skillet</h1>
          <p class="price"><data value="42.00">£42.00</data> <span class="price-note">incl. VAT</span></p>
          <p>Pre-seasoned and ready to use. Heavy enough to hold heat for a hard sear, and
            it goes from hob to oven.</p>

          <form class="buy" action="basket.html" method="post">
            <fieldset>
              <legend>Size</legend>
              <div class="options">
                <label><input type="radio" name="size" value="20" required> 20 cm</label>
                <label><input type="radio" name="size" value="24"> 24 cm</label>
                <label><input type="radio" name="size" value="28" checked> 28 cm</label>
              </div>
            </fieldset>
            <label class="qty" for="qty">Quantity</label>
            <input id="qty" name="qty" type="number" min="1" max="10" value="1" inputmode="numeric">
            <button type="submit">Add to basket</button>
          </form>

          <ul class="perks">
            <li>Free delivery over £40</li>
            <li>30-day returns</li>
          </ul>
        </div>
      </div>

      <section aria-labelledby="specs-heading">
        <h2 id="specs-heading">Specifications</h2>
        <dl class="specs">
          <div><dt>Diameter</dt><dd>28 cm (top rim)</dd></div>
          <div><dt>Depth</dt><dd>5 cm</dd></div>
          <div><dt>Material</dt><dd>Cast iron, vegetable-oil seasoning</dd></div>
          <div><dt>Hobs</dt><dd>Gas, electric, induction, oven</dd></div>
        </dl>
      </section>

      <section aria-labelledby="care-heading">
        <h2 id="care-heading">Care</h2>
        <ol class="care">
          <li>Wash with hot water and a brush; a little soap is fine.</li>
          <li>Dry straight away on a low heat.</li>
          <li>Wipe a thin film of oil over the surface while it's warm.</li>
        </ol>
      </section>

      <section aria-labelledby="related-heading">
        <h2 id="related-heading">Goes well with</h2>
        <ul class="cards">
          <li class="card">
            <img src="img/lid.png" width="600" height="400" loading="lazy" alt="">
            <h3><a href="lid.html">Glass lid, 28 cm</a></h3>
            <p>Fits the 28 cm skillet; steam vent.</p>
            <p class="card-price">£14.00</p>
          </li>
          <li class="card">
            <img src="img/trivet.png" width="600" height="400" loading="lazy" alt="">
            <h3><a href="trivet.html">Oak trivet with a routed groove to catch drips</a></h3>
            <p>Protects worktops from a hot pan.</p>
            <p class="card-price">£18.00</p>
          </li>
          <li class="card">
            <img src="img/spatula.png" width="600" height="400" loading="lazy" alt="">
            <h3><a href="spatula.html">Steel fish slice</a></h3>
            <p>Thin, flexible edge that slides under crusts without scratching seasoning.</p>
            <p class="card-price">£11.00</p>
          </li>
        </ul>
      </section>
    </main>

    <footer class="site-footer">
      <div class="container">
        <p><small>© 2026 Weeknight Kitchen Shop — a course exercise, not a real store.</small></p>
      </div>
    </footer>
  </body>
</html>
```

## Reference solution: the CSS

```css title="styles.css"
/* ---------- Tokens ---------- */
:root {
  color-scheme: light dark;
  --brand: oklch(48% 0.15 38);
  --brand-contrast: white;
  --bg: oklch(99% 0.004 80);
  --surface: white;
  --text: oklch(22% 0.01 60);
  --muted: oklch(45% 0.01 60);
  --border: oklch(88% 0.01 60);
  --radius: 0.5rem;
  --space: clamp(1rem, 0.75rem + 1vw, 1.5rem);
  --z-header: 20;
}

@media (prefers-color-scheme: dark) {
  :root {
    --brand: oklch(76% 0.12 50);
    --brand-contrast: oklch(20% 0.02 50);
    --bg: oklch(18% 0.005 60);
    --surface: oklch(23% 0.006 60);
    --text: oklch(93% 0.005 80);
    --muted: oklch(75% 0.01 80);
    --border: oklch(35% 0.01 60);
  }
}

/* ---------- Base ---------- */
*, *::before, *::after { box-sizing: border-box; }
img { max-width: 100%; height: auto; display: block; }
button, input { font: inherit; }

body {
  margin: 0;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  line-height: 1.55;
  color: var(--text);
  background: var(--bg);
}

h1, h2, h3 { line-height: 1.2; text-wrap: balance; }
h1 { font-size: clamp(1.75rem, 1.2rem + 2.2vw, 2.75rem); margin: 0 0 0.5rem; }
h2 { font-size: 1.5rem; margin: 2.5rem 0 1rem; }
h3 { font-size: 1.05rem; margin: 0; }

a { color: var(--brand); }
:focus-visible { outline: 3px solid var(--brand); outline-offset: 2px; }

.container { width: min(100% - 2rem, 72rem); margin-inline: auto; }

.skip-link { position: absolute; left: -999px; }
.skip-link:focus { left: 1rem; top: 1rem; z-index: calc(var(--z-header) + 1); background: var(--surface); padding: 0.5rem 1rem; }

/* ---------- Header ---------- */
.site-header {
  position: sticky;
  top: 0;
  z-index: var(--z-header);
  background: var(--surface);
  border-bottom: 1px solid var(--border);
}
.header-inner { display: flex; flex-wrap: wrap; align-items: center; gap: 0.5rem 1.5rem; padding-block: 0.75rem; }
.logo { font-weight: 700; text-decoration: none; color: var(--text); }
.logo span { color: var(--brand); }
.site-header nav { margin-inline-start: auto; }
.nav-list { display: flex; flex-wrap: wrap; gap: 1rem; list-style: none; margin: 0; padding: 0; }
[aria-current="page"] { font-weight: 700; }

/* ---------- Breadcrumb ---------- */
.breadcrumb ol { display: flex; flex-wrap: wrap; list-style: none; padding: 0; margin: 1rem 0; font-size: 0.9rem; }
.breadcrumb li + li::before { content: "/"; margin-inline: 0.5rem; color: var(--muted); }
.breadcrumb [aria-current="page"] { color: var(--text); text-decoration: none; font-weight: 400; }

/* ---------- Product: mobile first ---------- */
.product { display: grid; gap: var(--space); }

.gallery {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 0.5rem;
}
.gallery img { border-radius: var(--radius); }
.gallery-main { grid-column: 1 / -1; }

.price { font-size: 1.5rem; font-weight: 700; margin: 0 0 1rem; }
.price-note { font-size: 0.875rem; font-weight: 400; color: var(--muted); }

.buy { display: grid; grid-template-columns: max-content 5rem; align-items: center; gap: 0.75rem 1rem; margin-block: 1.5rem; }
.buy fieldset { grid-column: 1 / -1; border: 0; padding: 0; margin: 0; }
.buy legend { font-weight: 600; margin-bottom: 0.5rem; }
.options { display: flex; flex-wrap: wrap; gap: 0.5rem; }
.options label {
  display: flex; align-items: center; gap: 0.4rem;
  padding: 0.5rem 0.9rem;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  cursor: pointer;
}
.options label:has(:checked) { border-color: var(--brand); box-shadow: inset 0 0 0 1px var(--brand); }
.options input { accent-color: var(--brand); margin: 0; }
.qty { font-weight: 600; }
#qty { padding: 0.5rem; border: 1px solid var(--border); border-radius: var(--radius); background: var(--surface); color: var(--text); width: 100%; }

.buy button {
  grid-column: 1 / -1;
  padding: 0.85rem 1.5rem;
  border: 0;
  border-radius: var(--radius);
  background: var(--brand);
  color: var(--brand-contrast);
  font-weight: 700;
  cursor: pointer;
}
.buy button:hover { background: color-mix(in oklch, var(--brand), black 12%); }

.perks { color: var(--muted); padding-inline-start: 1.2rem; }

/* ---------- Specs ---------- */
.specs { display: grid; grid-template-columns: repeat(auto-fit, minmax(min(12rem, 100%), 1fr)); gap: 1rem; margin: 0; }
.specs div { padding: 1rem; background: var(--surface); border: 1px solid var(--border); border-radius: var(--radius); }
.specs dt { font-size: 0.8rem; text-transform: uppercase; letter-spacing: 0.05em; color: var(--muted); }
.specs dd { margin: 0; font-weight: 600; }

.care { max-width: 65ch; }

/* ---------- Related cards with subgrid ---------- */
.cards {
  list-style: none; padding: 0; margin: 0;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(15rem, 100%), 1fr));
  gap: 1.5rem;
}
.card {
  display: grid;
  grid-row: span 4;
  grid-template-rows: subgrid;
  gap: 0.5rem;
  padding-bottom: 1rem;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  overflow: hidden;
  position: relative;
}
.card > :not(img) { margin: 0; padding-inline: 1rem; }
.card h3 a { color: var(--text); text-decoration: none; }
.card h3 a::after { content: ""; position: absolute; inset: 0; }   /* whole card clickable */
.card:has(a:focus-visible) { outline: 3px solid var(--brand); outline-offset: 2px; }
.card h3 a:focus-visible { outline: none; }
.card-price { font-weight: 700; align-self: end; }

.site-footer { margin-top: 4rem; padding-block: 2rem; border-top: 1px solid var(--border); color: var(--muted); }

/* ---------- Wider screens ---------- */
@media (width >= 48rem) {
  .product { grid-template-columns: minmax(0, 1.2fr) minmax(0, 1fr); align-items: start; }
  .summary { position: sticky; top: 5rem; }
}

@media (width >= 64rem) {
  .gallery { grid-template-columns: 5rem minmax(0, 1fr); grid-template-rows: auto auto 1fr; }
  .gallery-main { grid-column: 2; grid-row: 1 / -1; }
  .gallery img:not(.gallery-main) { grid-column: 1; }
}

@media (prefers-reduced-motion: no-preference) {
  .card { transition: transform 150ms ease; }
  @media (hover: hover) {
    .card:hover { transform: translateY(-2px); }
  }
}
```

## Walkthrough of the interesting parts

**The gallery changes shape twice.** On phones it's a two-column grid where the main
photo spans both columns (`grid-column: 1 / -1`) and the two thumbnails sit side by side
below. From 64rem it becomes a narrow 5rem column of thumbnails beside the main photo: the
main image is moved to column 2 and spans all rows (`grid-row: 1 / -1`). The HTML never
changes, and the source order (main photo first) is the sensible reading order at every
size.

**The buy box sticks.** From 48rem, `.summary` is `position: sticky; top: 5rem`, so it
stays in view while the taller gallery scrolls past, then leaves with its section. The
grid uses `align-items: start` — without it, the summary would stretch to the full row
height and have no room to stick (the "needs room to move" rule from lesson 07).

**Radio buttons that look like buttons.** The inputs are still real radio buttons, so
arrow keys, form submission, `required` and screen-reader announcements all work. The
label is the visual button, and `label:has(:checked)` highlights the selected one —
`:has()` is covered properly in Level 3 · 07.

**The whole card is clickable — with one link.** The title link's `::after` pseudo-element
is absolutely positioned over the whole card (`inset: 0`, with the card as the containing
block). Clicking anywhere on the card follows the link, but assistive technology still
sees one link whose name is the product title — not a giant link containing the
description and price. The focus outline is moved from the link to the card with
`.card:has(a:focus-visible)`.

**Dark mode is nine lines.** Only the semantic variables change. The brand colour gets
*lighter* in dark mode (76% lightness instead of 48%) and the button text flips to a dark
colour, because a mid-tone orange doesn't have enough contrast against either black or
white text in both themes.

## Checking the result

html-validate (with the lesson-02 config) exited cleanly. We then loaded the page in
Chromium at three widths and both colour schemes, ran axe-core 4.13 (WCAG 2.x A/AA and
best practices), and measured the layout:

```text
360px  light: axe violations=0 scrollWidth=360  product cols "328px"             gallery "160px 160px"
800px  light: axe violations=0 scrollWidth=800  product cols "408px 340px"       gallery "200px 200px"
1280px light: axe violations=0 scrollWidth=1280 product cols "615.266px 512.719px" gallery "80px 527.266px"
1280px dark:  axe violations=0 scrollWidth=1280
360px  dark:  axe violations=0 scrollWidth=360
```

`scrollWidth` equal to the viewport width means no horizontal scroll at any size. We also
measured the top edge of each related card's price:

```text
360px:  [2154, 2527, 2905]   one card per row, so no alignment to do
800px:  [1570, 1570, 1570]   three cards in a row: prices aligned by subgrid
1280px: [1382, 1382, 1382]   cards grid "270px 270px 270px 270px" — auto-fill kept an empty 4th track
```

At 1280px there's room for four tracks and only three cards; because we used `auto-fill`,
the cards keep their 270px size instead of stretching to fill the row (compare lesson 06).
That's a design choice: switch to `auto-fit` and see which you prefer.

## How It Actually Works

Several independent layout systems are nested here, and each only controls its direct
children:

```text
body (block flow)
└─ main.container (block flow, width from min())
   ├─ .product (grid: 1 → 2 columns)
   │  ├─ .gallery (grid: 2 cols → rail + main)
   │  └─ .summary (block flow; sticky from 48rem)
   │     └─ form.buy (grid: label | input)
   │        └─ .options (flex, wrapping)
   ├─ dl.specs (grid: auto-fit)
   └─ ul.cards (grid: auto-fill)
      └─ li.card (grid: subgrid rows of the parent)
```

Choosing the layout mode per container — rather than one giant page grid — is what keeps
the CSS short: each container solves one problem, and changing one (say, the gallery at
64rem) can't disturb the others. The two exceptions that reach across levels are
deliberate: subgrid (cards share the parent's rows) and `position: sticky`/`absolute`,
which work against ancestors' boxes.

Theming works through inheritance. Every colour is read with `var()` at computed-value
time, and the `prefers-color-scheme` media query redefines the variables on `:root`. When
the OS theme changes, the query result flips, `:root`'s custom properties change, and
every element that inherits them recomputes its colours — no class toggling needed.

## Common mistakes

- **Sticky elements inside a stretched grid item** — no room to stick.
- **Custom-styled radio buttons that hide the input with `display: none`**, removing them
  from the keyboard and accessibility tree. Keep the input in the page (it can be visually
  subtle, but not removed).
- **Whole-card links wrapping all the content in `<a>`**, producing a very long link name.
- **Dark themes that only invert backgrounds**, leaving brand colours failing contrast.
- **`auto-fit` vs `auto-fill` chosen by accident.**

## Exercise

1. Build your own product page for something you own. Meet all nine requirements.
2. Test at 320px, 768px, 1024px and 1600px, and at 200% zoom.
3. Toggle dark mode in devtools (Rendering → prefers-color-scheme) and check every text
   colour's contrast with the colour picker.
4. Add a fourth related card with a very long title and confirm the prices still align.
5. Add a "Reviews" section with star ratings. Make the rating readable by screen readers
   (for example, visible text "4.5 out of 5") rather than stars alone.
