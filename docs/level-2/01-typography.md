---
description: "Set readable, fluid type: font stacks, @font-face and font-display, unitless line-height, clamp() for fluid sizes, measure, text-wrap and font features."
---

# 01 · Typography & Web Fonts

Most of what people do on the web is read. Typography — the choice of fonts, sizes,
spacing and line length — decides whether that reading is comfortable or tiring, and
it's also where CSS has quietly become excellent: fluid sizing with `clamp()`, balanced
headings with one property, variable fonts, and font loading you can control precisely.

## Font stacks

`font-family` takes a **list**. The browser uses the first font it has, character by
character, falling back down the list:

```css
body {
  font-family: "Inter", system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
}

code, pre, kbd {
  font-family: ui-monospace, "SF Mono", "Cascadia Code", Consolas, monospace;
}
```

- Quote names with spaces.
- **Always end with a generic family** — `serif`, `sans-serif`, `monospace`,
  `system-ui`, `ui-monospace`, `cursive` — so something sensible appears if nothing else
  matches.
- `system-ui` uses the operating system's interface font (San Francisco on Apple devices,
  Segoe UI on Windows, Roboto on Android). It's free, fast and looks native.

Fallback is **per character**: if your web font has no Telugu or Devanagari glyphs, those
characters are drawn from the next font in the stack that has them. That's why the
generic family at the end matters for multilingual sites.

## Loading web fonts with `@font-face`

```css title="styles.css"
@font-face {
  font-family: "Source Serif";
  src: url("/fonts/source-serif-4-variable.woff2") format("woff2");
  font-weight: 200 900;      /* a variable font covering this range */
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: "Source Serif";
  src: url("/fonts/source-serif-4-italic-variable.woff2") format("woff2");
  font-weight: 200 900;
  font-style: italic;
  font-display: swap;
}

body { font-family: "Source Serif", Georgia, serif; }
```

- **WOFF2** is the format to serve; every current browser supports it and it's
  well compressed.
- Each `@font-face` rule describes one **face** (a weight/style combination) of a family.
  When your CSS asks for `font-style: italic` and no italic face is declared, browsers
  *synthesise* a slanted version, which looks noticeably worse than a real italic.
- **Variable fonts** put a continuous range of weights (and sometimes widths or other
  axes) in one file — declare the range, then use any value: `font-weight: 650`.
- Fonts are only downloaded when text on the page actually uses them.
- Check the licence. Fonts from Google Fonts and many open-source foundries are free to
  self-host; commercial fonts often restrict web use.

### `font-display`: what happens while the font loads

| Value | Behaviour |
|-------|-----------|
| `auto` | Browser decides (usually like `block`) |
| `block` | Text invisible for up to ~3s, then fallback, swap when loaded |
| `swap` | Fallback immediately, swap whenever the font arrives |
| `fallback` | Very short invisible period (~100ms), short swap window (~3s), then stick with the fallback |
| `optional` | Very short invisible period; use the font only if it's already available (e.g. cached), otherwise keep the fallback for this page view |

`swap` guarantees text is readable immediately but can cause a visible reflow when the
font arrives; `optional` avoids reflow entirely at the cost of sometimes not showing the
web font. Level 4 · 02 shows how to reduce the reflow with fallback metric overrides
(`size-adjust` and friends) and when to preload fonts.

## Size

Set font sizes in `rem` so they respect the user's browser font setting (Level 1 · 09):

```css
html { font-size: 100%; }        /* = the user's default, usually 16px */
body { font-size: 1.0625rem; }   /* 17px at default settings */
h1   { font-size: 2.5rem; }
small, .meta { font-size: 0.875rem; }
```

### Fluid type with `clamp()`

`clamp(MIN, PREFERRED, MAX)` returns the preferred value, but never below MIN or above
MAX. Combining `rem` and `vw` in the preferred value gives type that grows smoothly with
the viewport:

```css
h1 { font-size: clamp(1.75rem, 1rem + 3vw, 3rem); }
```

We rendered that heading in Chromium at four viewport widths:

```text
320px  -> 28px     (1rem + 9.6px = 25.6px, below the 28px minimum, so clamped up)
768px  -> 39.04px  (16px + 23.04px)
1280px -> 48px     (16px + 38.4px = 54.4px, capped at the 3rem maximum)
1920px -> 48px
```

Always keep a `rem` part in the middle value. A pure `vw` size (`font-size: 4vw`) doesn't
respond to browser zoom or the user's font-size setting, which is an accessibility
failure (WCAG requires text to resize to 200%).

## Line height and measure

**Line height** for body text is usually 1.4–1.7; headings need less (1.1–1.25).

Write it **unitless**. The difference is in what inherits. We set line-height on two
parent `div`s with 16px text, one as `1.5em` and one as `1.5`, then measured a 32px `h2`
inside each:

```text
line-height: 1.5em on parent -> h2 line-height 24px
line-height: 1.5   on parent -> h2 line-height 48px
```

`1.5em` is computed to **24px on the parent**, and 24px is what inherits — so the 32px
heading's lines overlap. A unitless `1.5` inherits as the *number*, and each element
multiplies it by its own font size.

**Measure** is line length. Around 45–75 characters per line is comfortable for body
text; the `ch` unit makes this easy:

```css
.prose { max-width: 65ch; }
```

## Wrapping and hyphenation

```css
h1, h2, h3 { text-wrap: balance; }   /* even out line lengths in short blocks */
p          { text-wrap: pretty; }    /* avoid a single orphaned word on the last line */

.prose {
  hyphens: auto;            /* needs a correct lang attribute on the page */
  overflow-wrap: break-word;  /* break long URLs rather than overflowing */
}
```

`text-wrap: balance` is the fix for headlines with one lonely word on the second line.
Browsers limit it to short blocks (a handful of lines) because balancing is expensive.
`pretty` spends a little more effort on the last lines of paragraphs. Both were
supported (`CSS.supports()` returned `true`) in the Chromium 153 and WebKit 26.6 builds
we tested; where unsupported, text simply wraps normally.

## Other useful properties

```css
.caps     { text-transform: uppercase; letter-spacing: 0.06em; font-size: 0.8rem; }
.numbers  { font-variant-numeric: tabular-nums; }     /* aligned digits in tables */
.fraction { font-variant-numeric: diagonal-fractions; }
.ligs     { font-feature-settings: "liga" 1, "calt" 1; } /* low-level OpenType switches */
.clamp-3  {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;           /* truncate to 3 lines with an ellipsis */
}
```

The `-webkit-` line-clamp combination looks archaic but is the standardised,
cross-browser way to clamp lines today. Truncating text hides it from sighted users —
make sure the full text is reachable (a "Read more" link, or on the next page).

Prefer `font-variant-*` properties over `font-feature-settings` where one exists; the
low-level property overrides *all* features at once, so setting it in two places
clobbers the first.

## Worked example: a type system in a dozen lines

```css title="styles.css"
:root {
  --font-body: "Source Serif", Georgia, serif;
  --font-ui: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
}

body {
  font-family: var(--font-body);
  font-size: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  line-height: 1.6;
}

h1, h2, h3 {
  font-family: var(--font-ui);
  line-height: 1.15;
  text-wrap: balance;
}

h1 { font-size: clamp(2rem, 1.2rem + 3vw, 3.25rem); }
h2 { font-size: clamp(1.5rem, 1.1rem + 1.5vw, 2.125rem); }
h3 { font-size: 1.25rem; }

.prose { max-width: 65ch; }
.prose p { text-wrap: pretty; }
```

(`--font-body` is a custom property; lesson 09 covers them.) Body text grows gently from
16px on phones to 18px on large screens; headings scale more steeply; line length stays
readable at every width.

## How It Actually Works

Text layout happens in stages. First **font matching**: for each run of text, the browser
walks the `font-family` list and, within a family, picks the face closest to the requested
`font-weight`, `font-style` and `font-stretch` using the CSS font-matching algorithm. If
a face covers the requested weight range (variable fonts), it's used directly; otherwise
the nearest weight is chosen, and bold or italic may be synthesised.

Then **shaping**: a shaping engine (HarfBuzz in Chromium and Firefox; Core Text in Safari)
turns characters into positioned glyphs, applying kerning, ligatures and the
script-specific rules that languages such as Arabic and Telugu require — which is why you
should never try to "fix" letter spacing in those scripts manually.

Then **line breaking**: the browser finds break opportunities (spaces, hyphenation points
if `hyphens: auto`, anywhere if `overflow-wrap: anywhere`) and fills lines. The default is
a greedy algorithm — fit as many words as possible on each line. `text-wrap: balance` and
`pretty` swap in costlier algorithms that consider several lines at once.

Font *loading* interacts with layout: when a web font finishes loading, every piece of
text using it must be re-shaped and re-laid out, which is the reflow `font-display: swap`
can cause.

## Common mistakes

- **No generic family at the end** of a font stack.
- **Line heights in `px` or `em`** that break when font sizes change further down.
- **Pure `vw` font sizes** that ignore zoom.
- **Lines 120 characters long** on wide screens.
- **Loading six weights of a font** when the design uses two. Each is a separate download
  unless it's one variable font.
- **Faux bold/italic** because the corresponding face was never declared.
- **`hyphens: auto` without `lang`** — nothing happens.

## Exercise

1. Pick a free variable font (for example from Google Fonts), download the WOFF2 and
   self-host it with `@font-face`, including an italic face. Use it for your recipe page's
   body text.
2. Replace fixed heading sizes with `clamp()`. In devtools' device toolbar, drag the width
   between 320px and 1600px and watch the computed size.
3. Set `line-height: 1.5em` on `body` and look at your largest heading. Switch to `1.5`.
4. Add `text-wrap: balance` to headings and find a heading whose wrapping visibly
   improves.
5. Throttle the network to "Slow 3G" and compare `font-display: swap` with `optional`.
