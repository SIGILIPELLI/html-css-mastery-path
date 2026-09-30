---
description: "Make pages render fast: render-blocking CSS measured, critical CSS, preload and fetchpriority, fonts without layout shift, Core Web Vitals."
---

# 02 · Loading Performance: Critical CSS, Fonts & Layout Shift

How quickly a page *appears* is mostly decided by how HTML and CSS are delivered. CSS
blocks rendering, fonts can hide text or make it jump, images without dimensions push
content around — and Google measures all of it in the **Core Web Vitals**, which feed
into search ranking. This lesson measures the main effects and shows the fixes, all of
which are HTML and CSS decisions.

## Core Web Vitals in brief

| Metric | Measures | "Good" threshold (75th percentile of real visits) |
|--------|----------|---------------------------------------------------|
| **LCP** — Largest Contentful Paint | When the largest image or text block in the viewport rendered | ≤ 2.5 s |
| **CLS** — Cumulative Layout Shift | How much visible content moved unexpectedly | ≤ 0.1 |
| **INP** — Interaction to Next Paint | How quickly the page responds visually to clicks, taps and key presses | ≤ 200 ms |

LCP and CLS are strongly influenced by HTML and CSS; INP mostly by JavaScript and the
rendering work of lesson 01. Field data (from real users) is what counts; lab tools like
Lighthouse estimate it.

## CSS is render-blocking — measured

A stylesheet in the `<head>` blocks rendering until it has downloaded and parsed.
Otherwise the browser would paint unstyled content and then restyle it (a "flash of
unstyled content").

We served a tiny page and delayed its stylesheet by 1.5 seconds, then read the browser's
`first-contentful-paint` entry:

```html
<!-- blocking -->
<link rel="stylesheet" href="/big.css">
<!-- non-blocking (loads as print, switches to all when loaded) -->
<link rel="stylesheet" href="/big.css" media="print" onload="this.media='all'">
```

```text
no stylesheet:           first-contentful-paint = 36 ms
blocking stylesheet:     first-contentful-paint = 1532 ms   (repeat run: 1520 ms)
media="print" + onload:  first-contentful-paint = 16 ms     (repeat run: 24 ms)
```

The page's text sat invisible for the entire time the stylesheet was in flight. With the
`media="print"` trick, the browser downloads the file at low priority without blocking
(a print stylesheet isn't needed to render the screen), and the `onload` handler switches
it on — the page painted immediately, then restyled when the CSS arrived.

That's not a fix in itself: repainting with different styles 1.5 seconds later is a
worse experience than waiting. The real technique combines both.

## Critical CSS

1. **Inline** the CSS needed for the first screen ("above the fold") in a `<style>`
   element in the `<head>`. It arrives with the HTML — no extra request.
2. **Load the rest non-blocking** with the `media` swap.

```html
<head>
  <style>
    /* critical: reset, base typography, header, hero, layout of the first screen */
    *,*::before,*::after{box-sizing:border-box}
    body{margin:0;font-family:system-ui,sans-serif;line-height:1.55}
    .site-header{display:flex;align-items:center;gap:1rem;padding:1rem}
    .hero{min-height:60vh;display:grid;place-items:center}
  </style>
  <link rel="stylesheet" href="/css/site.css" media="print" onload="this.media='all'">
  <noscript><link rel="stylesheet" href="/css/site.css"></noscript>
</head>
```

The `<noscript>` fallback makes sure the full stylesheet still loads when JavaScript is
off. Tools can extract critical CSS automatically, but it adds build complexity — for
most sites, simpler wins come first:

- **Keep CSS small.** Delete unused rules; a 30 KB stylesheet on a fast connection is
  rarely the bottleneck.
- **One or two stylesheets**, not ten. Every blocking request is a round trip.
- **Serve it compressed and cached** (Brotli or gzip, long cache lifetimes with hashed
  filenames).
- **Don't use `@import`** in CSS files: the imported file isn't discovered until the
  importing file has downloaded and parsed, creating a sequential chain.
- **Use `media` on `<link>`** for stylesheets that only apply sometimes — `media="print"`,
  `media="(width >= 64rem)"` — they're still downloaded, but don't block rendering when
  they don't match.

## Resource hints

```html
<!-- start fetching something the page will definitely need, early -->
<link rel="preload" href="/fonts/source-serif-4.woff2" as="font" type="font/woff2" crossorigin>

<!-- make the LCP image the top priority -->
<img src="/img/hero.avif" alt="…" width="1600" height="900" fetchpriority="high">

<!-- open a connection to another origin in advance -->
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
```

- **`preload`** is for late-discovered resources that are definitely needed: fonts
  (discovered only after CSS is parsed and text is laid out), a background image used in
  the hero, a CSS file imported by another. Fonts need `crossorigin` even on the same
  origin (fonts are always fetched in CORS mode) or the preload is wasted. Preloading
  things that aren't needed early steals bandwidth from things that are.
- **`fetchpriority="high"`** on the hero image tells the browser it's the LCP element;
  `fetchpriority="low"` on below-the-fold carousels frees bandwidth.
- **Never lazy-load the LCP image** (Level 1 · 04).

## Fonts without layout shift

When a web font replaces a fallback font (`font-display: swap`), text reflows because the
two fonts have different metrics. You can make the fallback *match* the web font's
metrics with `@font-face` descriptors:

```css
@font-face {
  font-family: "Source Serif Fallback";
  src: local("Georgia");
  size-adjust: 104%;          /* scale Georgia's glyphs to match the web font's width */
  ascent-override: 92%;
  descent-override: 26%;
  line-gap-override: 0%;
}

body { font-family: "Source Serif", "Source Serif Fallback", Georgia, serif; }
```

The exact percentages depend on the two fonts and are computed from their metrics files —
tools such as the `fontaine` and `capsize` libraries, and framework font loaders, calculate
them for you. **Don't copy the numbers above**; they're illustrative. With well-matched
metrics, the swap still changes letter shapes but barely moves the layout.

Other font tactics:

- **Subset** fonts to the characters you use (for example, Latin only) with tools like
  `pyftsubset` or `glyphhanger`, and declare `unicode-range` so the browser only downloads
  a subset when it's needed.
- **Self-host** rather than loading from a third-party font CDN, so the font doesn't wait
  for a connection to another origin (and cross-site caching no longer exists in modern
  browsers anyway).
- **Fewer faces.** One variable font often replaces four or more static files.

## Preventing layout shift

CLS comes from content that appears or resizes after first render. The CSS/HTML
causes and fixes:

| Cause | Fix |
|-------|-----|
| Images and videos without dimensions | `width`/`height` attributes + `height: auto` (measured in Level 1 · 04: text jumped 266px without them) |
| Ads, embeds and iframes that load late | Reserve space with `min-height` or `aspect-ratio` on a wrapper |
| Web fonts swapping | Metric-matched fallbacks; `font-display: optional` for non-essential fonts |
| Content injected above existing content (banners, "cookie" bars) | Reserve space, overlay it (`position: fixed`), or insert below the viewport |
| Animations of layout properties | Animate `transform` instead — transforms aren't counted as layout shifts |

## Worked example: an optimised `<head>`

```html
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Roasted Tomato Soup — Weeknight Kitchen</title>

  <link rel="preload" href="/fonts/source-serif-4-var.woff2" as="font" type="font/woff2" crossorigin>

  <style>
    /* ~5 KB of critical CSS: tokens, reset, base type, header, hero */
  </style>

  <link rel="stylesheet" href="/css/site.9f3c1a.css" media="print" onload="this.media='all'">
  <noscript><link rel="stylesheet" href="/css/site.9f3c1a.css"></noscript>

  <script src="/js/app.4b2e7d.js" defer></script>
</head>
<body>
  …
  <img src="/img/soup-hero.avif" alt="…" width="1600" height="900" fetchpriority="high">
  …
</body>
```

Order matters: charset first (within 1024 bytes), then the critical path (font preload,
inline CSS), then non-blocking resources. The hashed filename (`site.9f3c1a.css`) lets the
server send `Cache-Control: max-age=31536000, immutable` — a changed file gets a new name.

## How It Actually Works

The HTML parser and the **preload scanner** discover resources as they read the
document. Each request gets a priority: render-blocking CSS in the `<head>` and
`fetchpriority="high"` images are fetched first; images are usually "low" until layout
reveals they're in the viewport; `media`-mismatched stylesheets are "lowest."

The browser won't run the first paint until every render-blocking stylesheet has loaded,
because the CSSOM must be complete for the cascade to be correct — a rule in the last
stylesheet could change anything. Scripts also interact with this: a classic
`<script>` placed after a stylesheet must wait for that stylesheet (the script might read
computed styles), which in turn blocks the parser. That's another reason for `defer`.

Layout shifts are computed per frame: for each element that moved and was visible in
both frames, the browser multiplies the fraction of the viewport affected (impact
fraction) by how far it moved relative to the viewport (distance fraction). CLS is the
largest burst ("session window") of those scores during the page's life. Shifts within
500ms of a user input are excluded, which is why expanding an accordion when clicked
doesn't count.

## Common mistakes

- **Many render-blocking stylesheets**, or CSS `@import` chains.
- **Loading all CSS non-blocking** without inlining critical CSS — a flash of unstyled
  content is worse than a short wait.
- **Preloading everything.**
- **Font preloads without `crossorigin`.**
- **Lazy-loading or low-prioritising the hero image.**
- **Images, ads and embeds without reserved space.**

## Exercise

1. Run Lighthouse (Chrome devtools → Lighthouse, mobile) on one of your project pages and
   note LCP, CLS and the "Eliminate render-blocking resources" opportunity.
2. In the Network panel, throttle to "Slow 4G", reload, and watch the filmstrip in the
   Performance panel. When does text first appear?
3. Inline your critical CSS and load the rest with the `media` swap. Measure again.
4. Add a web font with `font-display: swap`, observe the reflow on a throttled connection,
   then add a metric-adjusted fallback (use a tool to compute the values) and compare.
5. Find every image on your pages and confirm each has dimensions and appropriate
   `loading`/`fetchpriority`.
