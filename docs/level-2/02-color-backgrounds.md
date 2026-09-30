---
description: "Work with color in modern CSS: hex, rgb, hsl and oklch, color-mix and relative colors, WCAG contrast ratios computed by hand, backgrounds and gradients."
---

# 02 · Color, Backgrounds & Gradients

Colour in CSS used to mean hex codes copied from a design tool. Modern CSS can do much
more: it can mix colours, derive a hover shade from a brand colour, work in a colour
space where "lighter" actually looks lighter, and switch palettes for dark mode. It also
comes with a responsibility that's easy to measure and often ignored — **contrast**.

## Colour syntaxes

```css
.a { color: #b3401a; }                    /* hex: RRGGBB */
.b { color: #b3401a80; }                  /* hex with alpha: RRGGBBAA */
.c { color: rgb(179 64 26); }             /* space-separated modern syntax */
.d { color: rgb(179 64 26 / 50%); }       /* with alpha */
.e { color: hsl(15 75% 40%); }            /* hue, saturation, lightness */
.f { color: oklch(55% 0.18 40); }         /* lightness, chroma, hue */
.g { color: currentColor; }               /* the element's own `color` value */
.h { color: transparent; }
```

The comma syntax `rgb(179, 64, 26)` and `rgba()` still work everywhere; the space syntax
with `/ alpha` is the modern form and is what newer functions use.

### Why `oklch`

HSL looks intuitive — hue, saturation, lightness — but its "lightness" isn't how bright a
colour *looks*. `hsl(60 100% 50%)` (yellow) and `hsl(240 100% 50%)` (blue) have the same
L value and wildly different perceived brightness. That makes palettes built by varying
HSL lightness uneven, and makes contrast unpredictable.

**OKLCH** is designed around perception:

- **L** — perceived lightness, 0% (black) to 100% (white).
- **C** — chroma, how colourful (0 is grey; practical values rarely exceed ~0.37).
- **H** — hue angle, 0–360.

Changing L while holding C and H gives shades that look evenly spaced, and two colours
with the same L look roughly equally bright. OKLCH can also express colours outside the
sRGB range for wide-gamut displays; browsers map them back to what the screen can show.

## Mixing and deriving colours

```css
:root { --brand: #b3401a; }

.tint       { background: color-mix(in oklch, var(--brand) 20%, white); }
.hover      { background: color-mix(in oklch, var(--brand), black 15%); }
.translucent { background: rgb(from var(--brand) r g b / 0.5); }
.lighter    { color: oklch(from var(--brand) calc(l + 0.15) c h); }
```

`color-mix()` blends two colours in a colour space you choose. The **relative colour
syntax** (`rgb(from ...)`, `oklch(from ...)`) takes a colour apart into channels you can
reuse or adjust with `calc()`.

The mixing space matters. We computed several values in Chromium and read them back:

```text
oklch(55% 0.18 40)                          -> oklch(0.55 0.18 40)
color-mix(in srgb,  #b3401a 50%, white)     -> color(srgb 0.85098 0.62549 0.55098)
color-mix(in oklch, #b3401a 50%, white)     -> oklch(0.763927 0.0787013 37.4478)
hsl(15 75% 40%)                             -> rgb(179, 64, 26)
#b3401a80                                   -> rgba(179, 64, 26, 0.5)
rgb(from #b3401a r g b / 0.5)               -> color(srgb 0.701961 0.25098 0.101961 / 0.5)
```

Two things to notice. First, computed colours are **not always `rgb()`** any more — values
from modern colour functions are serialised in their own space, which matters if
JavaScript code parses `getComputedStyle(...).color` expecting `rgb(...)`. Second, the
two 50% mixes are different colours: sRGB mixing interpolates the gamma-encoded channels,
OKLCH mixing interpolates perceived lightness and hue. For tints and shades, `oklch`
usually looks more natural; `srgb` matches what older tools produced.

All of these functions were supported in both Chromium 153 and WebKit 26.6 in our
testing.

## Contrast

WCAG defines a **contrast ratio** from 1:1 (identical) to 21:1 (black on white), based
on the relative luminance of the two colours. The AA requirements:

| Content | Minimum ratio (AA) | AAA |
|---------|--------------------|-----|
| Body text | **4.5:1** | 7:1 |
| Large text (≥ 24px, or ≥ 18.66px bold) | **3:1** | 4.5:1 |
| UI components and meaningful graphics (borders of inputs, icons, focus rings) | **3:1** | — |

We implemented the WCAG formula (convert each sRGB channel to linear light, weight them
0.2126/0.7152/0.0722, then `(L1 + 0.05) / (L2 + 0.05)`) and ran it on some colours:

```text
#767676 on #ffffff: 4.54:1   <- the lightest grey that passes AA on white
#777777 on #ffffff: 4.48:1   <- one step lighter: fails
#999999 on #ffffff: 2.85:1   <- common "subtle" grey: fails badly
#b3401a on #ffffff: 5.73:1
#8a2f12 on #fffdf9: 8.28:1   <- the recipe project's headings
#555555 on #fffdf9: 7.34:1   <- the recipe project's secondary text
#1a5fb4 on #ffffff: 6.29:1   <- the focus ring colour
```

The pass/fail line is sharp: `#767676` passes and `#777777` fails. Don't eyeball it —
devtools shows the contrast ratio in the colour picker for any `color` value, and axe
checks text contrast automatically.

Contrast isn't the only rule: **don't use colour alone to convey information.** A red
border on an invalid field needs an error message too; a chart's lines need labels or
patterns, not just hues.

## Backgrounds

```css
.hero {
  background-color: #2b1a12;                   /* shows while the image loads, and behind transparency */
  background-image: url("img/kitchen.jpg");
  background-size: cover;                      /* fill the box, cropping as needed */
  background-position: center 30%;
  background-repeat: no-repeat;
}

/* the same in one shorthand: position / size come together */
.hero { background: #2b1a12 url("img/kitchen.jpg") center 30% / cover no-repeat; }
```

`cover` fills the box and crops; `contain` fits the whole image and leaves gaps. The
shorthand resets every background sub-property you don't mention, so put it before
any longhands you add.

Background images are **decoration**: they have no `alt`, aren't in the accessibility
tree, and aren't printed by default. If the image carries meaning, use an `<img>`.

### Text over images

Text on a photo fails contrast somewhere unless you control it. Layer a gradient over
the image — multiple backgrounds stack with the **first on top**:

```css
.hero {
  color: white;
  background:
    linear-gradient(to top, rgb(0 0 0 / 0.75), rgb(0 0 0 / 0.1) 60%),
    url("img/kitchen.jpg") center / cover;
}
```

## Gradients

Gradients are images generated by the browser, so they go anywhere an image can:

```css
.linear  { background: linear-gradient(135deg, #ffecd2, #fcb69f); }
.stops   { background: linear-gradient(to right, #b3401a 0 30%, #e0b457 30% 60%, #2fa98d 60%); } /* hard stops = stripes */
.radial  { background: radial-gradient(circle at 20% 20%, #fff8, transparent 40%), #b3401a; }
.conic   { background: conic-gradient(#b3401a 0 72%, #eadfd3 0); border-radius: 50%; } /* 72% pie chart */
.oklab   { background: linear-gradient(in oklch, #b3401a, #2fa98d); }
```

Two stops at the same position create a hard edge, which is how stripes and simple pie
charts are made. Gradients between saturated colours in sRGB can pass through a muddy
grey in the middle; `in oklch` (or `in oklab`) interpolation keeps the midpoint vivid.

## Shadows and opacity

```css
.card    { box-shadow: 0 1px 2px rgb(0 0 0 / 0.08), 0 4px 12px rgb(0 0 0 / 0.08); }
.inset   { box-shadow: inset 0 0 0 2px #b3401a; }  /* a "border" that takes no space */
.heading { text-shadow: 0 1px 2px rgb(0 0 0 / 0.4); }
.faded   { opacity: 0.6; }                          /* affects the element AND its children */
.glass   { background: rgb(255 255 255 / 0.7); backdrop-filter: blur(8px); }
```

`opacity` fades everything including text; to fade only the background, use an alpha
colour on `background` instead. `opacity` below 1 also creates a stacking context — the
cause of a nasty z-index bug in lesson 07.

## Worked example: a palette from one brand colour

```css title="styles.css"
:root {
  --brand: oklch(52% 0.16 38);
  --brand-hover: oklch(from var(--brand) calc(l - 0.08) c h);
  --brand-tint: oklch(from var(--brand) 95% 0.03 h);
  --text: oklch(25% 0.02 38);
  --muted: oklch(45% 0.02 38);
  --surface: oklch(99% 0.005 80);
  --border: oklch(88% 0.02 60);
}

a            { color: var(--brand); }
a:hover      { color: var(--brand-hover); }
.notice      { background: var(--brand-tint); border-left: 4px solid var(--brand); }
body         { color: var(--text); background: var(--surface); }
.meta        { color: var(--muted); }
```

Every derived colour keeps the brand's hue and changes only lightness or chroma, so the
palette hangs together. After building a palette like this, check each text/background
pair's contrast in devtools — perceptual lightness makes contrast *more predictable*,
not automatically compliant.

## How It Actually Works

A CSS colour is a point in a **colour space**. `rgb()`, hex and `hsl()` all describe
**sRGB**, the space most screens were built around. sRGB values are *gamma-encoded*:
channel value 128 is not half the light of 255 (it's roughly a fifth). That's why the WCAG
formula first converts channels to **linear light**, and why mixing in sRGB gives
different results from mixing in a linear or perceptual space.

**OKLab** (and its polar form **OKLCH**) is a space fitted to human perception data, so
equal steps look equal. When you write `color-mix(in oklch, A, B)`, the browser converts A
and B to OKLCH, interpolates each channel (hue along the shorter arc by default), and
produces a colour in that space. At paint time, every colour is converted to the display's
colour space; colours outside the display's gamut are mapped inside it.

Gradients are painted by the same machinery: each pixel's colour is an interpolation
between stops in the gradient's interpolation space (sRGB by default for legacy syntax
compatibility, or whatever you name with `in`).

## Common mistakes

- **Light grey text** (`#999` on white) that fails contrast.
- **Assuming computed colours are always `rgb()`** in JavaScript.
- **Meaning carried by colour alone.**
- **Text over photos** with no overlay.
- **Using `opacity` to fade a background** and fading the text with it.
- **Background images for content** — no alt text, invisible to assistive tech.
- **Hard-coding every shade** instead of deriving them from a few custom properties.

## Exercise

1. Pick one brand colour and build the palette from the worked example. Apply it to your
   recipe page.
2. Check the contrast of every text colour against its background using the devtools
   colour picker. Fix any below 4.5:1.
3. Build a hero section with a background photo, a gradient overlay and white text. Check
   contrast at the lightest part of the image.
4. Make a CSS-only progress ring for "72% complete" with `conic-gradient`, and put the
   text "72% complete" inside it for screen readers and everyone else.
5. Compare `linear-gradient(to right, blue, yellow)` with `linear-gradient(in oklch to right,
   blue, yellow)`. Describe the difference in the middle.
