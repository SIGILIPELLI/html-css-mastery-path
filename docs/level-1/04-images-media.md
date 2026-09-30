---
description: "Add images, video and audio properly: writing alt text, width and height to prevent layout shift, srcset and sizes, picture, figure and lazy loading."
---

# 04 · Images & Media

Images are usually the heaviest part of a page, the most common cause of content jumping
around while it loads, and — when their text alternatives are missing — the most common
accessibility failure on the web. The good news is that HTML gives you everything you
need to handle all three, and none of it requires JavaScript.

## The `<img>` element

```html
<img src="images/tomato-soup.jpg"
     alt="A bowl of tomato soup topped with basil and cream"
     width="1200" height="800">
```

Three attributes you should write every time:

- **`src`** — the image URL.
- **`alt`** — the text alternative. Required.
- **`width` and `height`** — the image's intrinsic dimensions in pixels (or at least
  the right *ratio*). Explained below; this is the one people skip.

### Writing `alt` text

`alt` replaces the image for anyone who can't see it: screen-reader users, people on
slow connections where the image failed, search engines. Write what the image
*communicates in this context*:

| Situation | Good `alt` |
|-----------|-----------|
| Photo that illustrates the content | `alt="A bowl of tomato soup topped with basil"` |
| Image that is a link | Describe the destination: `alt="Home"` for a logo linking home |
| Chart | The takeaway: `alt="Sales doubled between March and June"` — and put the data in a table or text nearby |
| Image of text | The text itself |
| Purely decorative (a flourish, a background texture) | `alt=""` — empty, not missing |

**Empty `alt=""` and a missing `alt` are different.** Empty tells assistive tech "skip
this." Missing makes many screen readers fall back to reading the *file name* —
"IMG underscore 4 0 2 7 dot J P G."

Don't start with "Image of…" or "Picture of…" — the role is already announced.

## `width` and `height`: reserving space

When the browser parses an `<img>`, it doesn't know how big the image is until the file
starts downloading. Without dimensions it lays the page out with a zero-height image,
then re-lays everything out when the image arrives, pushing content down. That jump is
measured by Google's **Cumulative Layout Shift** metric, and users experience it as
tapping the wrong thing.

We measured it. A page with two images — one without dimensions, one with
`width="800" height="533"` — and a paragraph after them, loaded in a 400px-wide Chromium
window. We held back the image responses and measured, then released them:

```css
img { max-width: 100%; height: auto; display: block; }
```

```text
before images arrive -> n: 0x0, w: 400x267, text top: 283
after images load    -> n: 400x267, w: 400x267, text top: 549
```

The image **with** dimensions (`w`) had its 400×267 box reserved immediately, scaled down
from 800×533 because of `max-width: 100%`. The one **without** (`n`) was 0×0 until it
loaded, and then the text below jumped 266 pixels.

The trick that makes this work responsively: browsers turn `width` and `height`
attributes into a CSS `aspect-ratio`, and `height: auto` then uses that ratio. So you
should **always** pair the attributes with this CSS:

```css title="styles.css"
img, video {
  max-width: 100%;
  height: auto;
}
```

## Responsive images: `srcset` and `sizes`

A 1600px-wide photo is right for a high-density laptop screen and wasteful for a phone.
`srcset` lets you offer several files and lets the browser pick.

### Width descriptors (`w`) with `sizes`

```html
<img src="img/photo-800.png"
     srcset="img/photo-400.png 400w,
             img/photo-800.png 800w,
             img/photo-1600.png 1600w"
     sizes="(min-width: 900px) 600px, 100vw"
     alt="…" width="800" height="533">
```

- `srcset` lists files and their **actual pixel widths**.
- `sizes` tells the browser how wide the image will be **displayed**: "600 CSS pixels
  wide when the viewport is at least 900px, otherwise the full viewport width." The
  browser needs this because it chooses the file before CSS layout has happened.
- `src` is the fallback for very old browsers.

### Density descriptors (`x`)

For images displayed at a fixed size (logos, avatars), describe pixel density instead:

```html
<img src="img/photo-400.png"
     srcset="img/photo-400.png 1x, img/photo-800.png 2x"
     width="400" height="267" alt="…">
```

### What the browser actually chose

We loaded both images above at different viewport widths and device pixel ratios (DPR)
in Chromium and read each image's `currentSrc`:

```text
viewport 375  dpr 1: ["photo-400.png","photo-400.png"]
viewport 375  dpr 2: ["photo-800.png","photo-800.png"]
viewport 375  dpr 3: ["photo-1600.png","photo-800.png"]
viewport 1280 dpr 1: ["photo-800.png","photo-400.png"]
viewport 1280 dpr 2: ["photo-1600.png","photo-800.png"]
```

Read the first column (the `w` image): on a 375px phone at DPR 3, the image displays at
375 CSS pixels × 3 = 1125 device pixels, so the browser took the 1600w file. On a
1280px laptop at DPR 1, `sizes` says 600px, so 800w is enough. The `x` image never
exceeds 2x because that's the largest candidate offered.

The spec lets browsers pick differently — for example, a browser may choose a smaller
file on a slow connection or reuse a larger one already in cache. Treat `srcset` as
*hints*, not commands.

## `<picture>`: art direction and formats

Use `<picture>` when you need **control** rather than hints:

```html
<picture>
  <!-- different crop on narrow screens (art direction) -->
  <source media="(max-width: 600px)" srcset="img/soup-square.avif" type="image/avif">
  <source media="(max-width: 600px)" srcset="img/soup-square.jpg">
  <!-- modern format with fallback -->
  <source srcset="img/soup-wide.avif" type="image/avif">
  <img src="img/soup-wide.jpg" alt="A bowl of tomato soup" width="1200" height="600">
</picture>
```

The browser takes the **first** `<source>` whose `media` matches and whose `type` it
supports. The `<img>` is mandatory: it's what actually renders (the `<source>`s only
change its URL), and it carries `alt`, `width`, `height` and your CSS classes.

AVIF and WebP are supported by all current major browsers, so the `type` fallback is
mostly for older ones. JPEG remains a safe universal fallback.

## Lazy loading and decoding

```html
<img src="…" alt="…" width="800" height="533" loading="lazy">
```

`loading="lazy"` defers images that are far below the viewport until the user scrolls
near them. Use it for images **below the fold**. Don't use it on the main image at the
top of the page — delaying the most important image makes the page feel slower.
Level 4 · 02 covers `fetchpriority="high"` for that hero image.

## `<figure>` and `<figcaption>`

When an image (or code sample, chart, quote) is referenced from the text and has a
caption, wrap it:

```html
<figure>
  <img src="img/roasting.jpg" alt="Halved tomatoes on a baking tray, skin side up"
       width="1200" height="800">
  <figcaption>Roast skin-side up so the juices stay in the tray.</figcaption>
</figure>
```

The caption doesn't replace `alt`. They do different jobs: `alt` stands in for the
image; the caption adds to it for everyone.

## Video and audio

```html
<video controls width="1280" height="720" poster="img/video-poster.jpg" preload="metadata">
  <source src="media/blending.webm" type="video/webm">
  <source src="media/blending.mp4" type="video/mp4">
  <track kind="captions" src="media/blending.en.vtt" srclang="en" label="English" default>
  <p>Your browser can't play this video. <a href="media/blending.mp4">Download it</a>.</p>
</video>

<audio controls src="media/interview.mp3"></audio>
```

- `controls` gives you the browser's accessible player UI. Without it there's no way to
  play or pause unless you build controls yourself.
- `<track kind="captions">` adds captions from a WebVTT file. Captions are essential for
  deaf and hard-of-hearing viewers and anyone watching with the sound off.
- `preload="metadata"` fetches duration and dimensions only.
- Autoplay is only permitted by browsers when the video is `muted`; even then, give the
  user a way to stop it.

A minimal WebVTT file:

```text title="media/blending.en.vtt"
WEBVTT

00:00:00.000 --> 00:00:03.500
Let the soup cool for five minutes first.

00:00:03.500 --> 00:00:07.000
Blend in batches, never more than half full.
```

## Inline SVG

Icons and simple illustrations are often best as SVG — they're text, scale perfectly
and can be styled with CSS. For a decorative icon next to text:

```html
<button type="button">
  <svg aria-hidden="true" width="16" height="16" viewBox="0 0 16 16">
    <path d="M2 8h12M8 2v12" stroke="currentColor" stroke-width="2"/>
  </svg>
  Add ingredient
</button>
```

`aria-hidden="true"` hides it from assistive tech because the button text already says
everything. `stroke="currentColor"` makes it follow the text colour.

## How It Actually Works

**Image selection** happens in the HTML parser's world, not the layout engine's. When the
preload scanner sees an `<img>`, it must pick a URL immediately to start the download —
before any CSS has been applied and before layout knows how wide the image will be.
That's why `sizes` exists: it's you telling the browser the layout answer in advance.
The algorithm is: evaluate `sizes` to get a display width in CSS pixels, multiply by the
device pixel ratio, divide each `w` candidate by the display width to get its effective
density, and pick the candidate whose density is the best fit (usually the smallest one
that is at least the DPR).

**Aspect-ratio mapping** is defined in the HTML spec's rendering section: `width` and
`height` attributes on `img` map to the CSS declaration
`aspect-ratio: auto <width> / <height>`. The `auto` means "use the real ratio once the
image loads, this one until then." Combined with `height: auto`, the box has the right
shape from the first layout pass.

**Decoding** — turning compressed bytes into a bitmap — is expensive for large images and
happens off the main thread in modern browsers. That's one reason serving a 1600px image
to a 400px phone costs more than just bandwidth.

## Common mistakes

- **Missing `alt`**, or alt text that describes the file ("hero-image-final.jpg").
- **Omitting `width`/`height`** because "CSS sets the size anyway." CSS sizing doesn't
  help until the image loads; the attributes do.
- **`srcset` without `sizes`** for `w` descriptors — the browser assumes `100vw`, and
  downloads a much bigger file than needed for a small thumbnail.
- **`loading="lazy"` on the largest, top-of-page image.**
- **Putting `alt` on `<source>`** in `<picture>` — it belongs on the `<img>`.
- **Video without captions**, or autoplaying video with sound.

## Exercise

1. Add three images to your recipe page from lesson 03: one informative photo, one
   decorative divider (`alt=""`), and one image that is a link. Write appropriate `alt`
   for each.
2. Add the `img { max-width: 100%; height: auto; }` rule. In devtools, throttle the
   network to "Slow 3G", reload, and watch the page with and without `width`/`height`
   on the photo.
3. Export one photo at 400, 800 and 1600 pixels wide. Add `srcset` and `sizes`. Using
   the device toolbar at different widths and DPRs, type `$0.currentSrc` in the console
   with the image selected and record which file loads.
4. Wrap a photo in `<figure>` with a caption. Then write a short WebVTT file and attach
   it to any short video you have.
