---
description: "Layouts for any language: dir and lang, logical properties measured in LTR and RTL, writing modes, text expansion, other scripts, bidi text."
---

# 05 · Logical Properties, Writing Modes & Internationalization

Hundreds of millions of people read right-to-left scripts — Arabic, Hebrew, Persian,
Urdu — and a site that serves India alone will meet Telugu, Hindi, Tamil and
Urdu within the same audience. CSS written with `left` and `right` assumes one direction.
**Logical properties** describe layout in terms of the text's own flow instead, so one
stylesheet works for every direction and writing mode. Combined with correct `lang` and
`dir` attributes, they're most of what HTML and CSS need for internationalisation.

## Direction and language in HTML

```html
<html lang="ar" dir="rtl">
```

- `lang` (Level 1 · 02) sets the language for screen-reader pronunciation, hyphenation
  and font selection.
- `dir` sets the base direction: `ltr`, `rtl` or `auto` (decide from the first strong
  character in the content — useful for user-generated text).

Set direction with the `dir` **attribute**, not the CSS `direction` property. Direction is
content meaning — it affects the order of table columns, how bidirectional text is
reordered, and how assistive technology reads — and markup is where meaning lives.

## Physical vs logical

Physical properties refer to the screen: top, right, bottom, left. Logical properties
refer to the flow of text:

- **inline** axis — the direction text runs within a line (left→right in English,
  right→left in Arabic, top→bottom in vertical Japanese).
- **block** axis — the direction lines and paragraphs stack.
- **start / end** — the beginning and end along each axis.

| Physical | Logical |
|----------|---------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` / `margin-bottom` | `margin-block-start` / `margin-block-end` |
| `margin-left` + `margin-right` | `margin-inline` |
| `padding-top` + `padding-bottom` | `padding-block` |
| `border-left` | `border-inline-start` |
| `width` / `height` | `inline-size` / `block-size` |
| `max-width` | `max-inline-size` |
| `top` / `left` (positioning) | `inset-block-start` / `inset-inline-start` |
| `text-align: left` | `text-align: start` |
| `float: left` | `float: inline-start` |
| `border-top-left-radius` | `border-start-start-radius` |

## Measured: the same CSS in LTR and RTL

We rendered one stylesheet in a 1280px window, once with `dir="ltr"` and once with
`dir="rtl"`:

```css
.a { margin-inline-start: 40px; padding-inline-end: 10px;
     border-inline-start: 4px solid; text-align: start; width: 200px; }
.b { margin-left: 40px; width: 200px; }
.c { display: flex; gap: 10px; }   /* two 50px items */
```

```text
ltr: .a margin-left 40px, margin-right 0px, border-left 4px, border-right 0px
     .a left edge = 40      .b left edge = 40      first flex item left = 0
rtl: .a margin-left 0px, margin-right 40px, border-left 0px, border-right 4px
     .a left edge = 1026    .b left edge = 1080    first flex item left = 1230
```

In RTL, the logical `.a` flipped completely: its margin and border moved to the right
side, and the box sits 40px from the right edge (1280 − 40 − 214 = 1026). The physical
`.b` did **not** mirror — and worse, its `margin-left: 40px` had no visible effect,
because in an RTL block the box is placed against the right edge and the left margin is
simply the leftover space. Flexbox and Grid are already logical: the first flex item moved
to the right automatically.

That's the general picture: **Flexbox, Grid, `text-align: start` and logical properties
mirror for free; anything written with left/right has to be overridden per direction.**
Write logical properties by default, and physical ones only when something is genuinely
physical (a shadow that should always fall to the bottom-right, say).

## Things that shouldn't mirror

Some content keeps its direction in RTL:

- **Icons that mean a direction in time or reading** — "next" and "back" arrows *should*
  flip. Icons of real objects (a clock, a magnifying glass, a checkmark) usually shouldn't.
  Flip selectively:

    ```css
    [dir="rtl"] .icon-arrow { scale: -1 1; }
    /* or, relative to the element's own direction: */
    .icon-arrow:dir(rtl) { scale: -1 1; }
    ```

- **Numbers, code, phone numbers, URLs** stay left-to-right inside RTL text; the Unicode
  bidirectional algorithm handles most of this automatically.
- **Media controls** (play, progress bar) conventionally don't mirror.

## Bidirectional text

When RTL and LTR text mix — an English product name in an Arabic sentence, or an Arabic
username on an English page — the browser applies the Unicode Bidirectional Algorithm.
It usually gets it right, but user-supplied text can "leak" its direction into
surrounding punctuation. Isolate it:

```html
<p>Posted by <bdi>مريم</bdi>: 3 comments</p>
<p>Search results for <span dir="auto">{{ query }}</span></p>
```

`<bdi>` and `dir="auto"` isolate the inner text so its direction doesn't reorder the
punctuation and numbers around it. Use them whenever you insert text whose direction you
don't know.

## Writing modes

`writing-mode` changes the block and inline axes themselves — for vertical Chinese,
Japanese, Korean or Mongolian text, or for design effects such as vertical table headers:

```css
.vertical { writing-mode: vertical-rl; inline-size: 200px; padding-block-start: 10px; }
```

We measured a `vertical-rl` element with `inline-size: 200px`:

```text
rendered width 32, height 200; padding-right 10px, padding-top 0px
```

In a vertical writing mode, the inline axis is vertical — so `inline-size: 200px` became
the **height**, and the block axis runs right to left, so `padding-block-start` became
**padding-right**. Logical properties followed the text; physical ones would have been
wrong.

## Designing for text expansion and other scripts

- **Text length varies.** A German or Finnish translation is often much longer than the
  English; Chinese and Japanese are often shorter. Avoid fixed widths on buttons, tabs and
  navigation — let them wrap or grow — and test with a long pseudo-translation.
- **Line height and glyph height vary.** Scripts like Telugu, Devanagari, Thai and
  Arabic have taller ascenders and descenders or stacked marks. A `line-height` tuned for
  Latin text can clip them; 1.5 or more for body text is safer, and avoid
  `overflow: hidden` on single-line containers of translated text.
- **Fonts.** Your Latin web font almost certainly doesn't include Telugu or Arabic glyphs.
  Put a script-specific font in the stack (for example the Noto families) and use
  `unicode-range` so it only downloads when needed:

    ```css
    @font-face {
      font-family: "Noto Sans Telugu";
      src: url("/fonts/NotoSansTelugu-var.woff2") format("woff2");
      unicode-range: U+0C00-0C7F;   /* the Telugu block */
      font-display: swap;
    }
    body { font-family: "Inter", "Noto Sans Telugu", system-ui, sans-serif; }
    ```

- **Don't letter-space or uppercase connected scripts.** `letter-spacing` breaks the
  joining of Arabic letters; `text-transform: uppercase` means nothing in most non-Latin
  scripts. Scope such styles with `:lang(en)` if needed.
- **Emphasis conventions differ.** Italics don't exist in many scripts; `:lang()` lets you
  choose alternatives:

    ```css
    em:lang(ja), em:lang(zh) { font-style: normal; text-emphasis: filled dot; }
    ```

- **Quotes, dates, numbers** — use the platform: `<q>` with the right `lang`, and format
  dates and numbers with `Intl.DateTimeFormat`/`Intl.NumberFormat` in JavaScript rather
  than hard-coding "MM/DD/YYYY".

## Worked example: a direction-agnostic media object

```css
.media {
  display: flex;
  gap: var(--space-3);
  align-items: start;
}

.media__image {
  flex-shrink: 0;
  inline-size: 4rem;
  border-radius: var(--radius-md);
}

.media__body {
  border-inline-start: 3px solid var(--color-brand);
  padding-inline-start: var(--space-3);
  text-align: start;
}

.media__more::after {
  content: "→";
  margin-inline-start: 0.25em;
  display: inline-block;
}
.media__more:dir(rtl)::after { scale: -1 1; }   /* the arrow points the reading direction */
```

With `dir="rtl"` on the page (or on one `.media` element), the image moves to the right,
the accent border and padding move to the body's right edge, the text aligns right and
the arrow points left — with no RTL-specific overrides except the arrow.

## How It Actually Works

Every box has a **writing mode** (`horizontal-tb`, `vertical-rl`, `vertical-lr`) and a
**direction** (`ltr`, `rtl`) — together, its *writing-mode context*. When the browser
computes styles, each logical property is mapped to its physical equivalent **for that
element's context**: in `horizontal-tb` + `rtl`, `margin-inline-start` maps to
`margin-right`. The mapping happens during the cascade, so if an element has both
`margin-left` and `margin-inline-start`, whichever comes later in the cascade wins —
they're two names for the same underlying property.

Layout algorithms themselves are written in logical terms in the specs: block layout
stacks boxes along the block axis, inline layout fills lines along the inline axis, and
Flexbox's `row` means "the inline direction." That's why those layouts mirror
automatically while hand-positioned elements don't.

Bidi reordering happens at line layout: text is split into runs by direction, and the
Unicode Bidirectional Algorithm decides the visual order of runs within each line.
`<bdi>`, `dir="auto"` and `unicode-bidi: isolate` create isolation boundaries so a run's
direction can't affect its neighbours.

## Common mistakes

- **`left`/`right` everywhere**, then a separate RTL stylesheet full of overrides.
- **`direction: rtl` in CSS** instead of the `dir` attribute.
- **Mirroring every icon**, including ones that shouldn't flip.
- **Fixed-width buttons and tabs** that break in longer languages.
- **Line heights that clip** taller scripts.
- **Letter-spacing Arabic** or uppercasing non-Latin text.
- **User-generated text without isolation**, scrambling punctuation around it.

## Exercise

1. Convert your Level 2 product page to logical properties (search for `left`, `right`,
   `top`, `bottom`, `width`, `height` in the CSS). Add `dir="rtl"` to `<html>` and fix
   anything that doesn't mirror.
2. Decide which icons on your pages should flip in RTL, and implement it with `:dir(rtl)`.
3. Replace your navigation labels with a long pseudo-translation (for example, double
   every word). Fix every overflow.
4. Add a paragraph of Telugu or Hindi text (copy a sentence from a news site in that
   language) and check the line height and font fallback in devtools' Computed → Rendered
   Fonts section.
5. Build a comment list where usernames are inserted in `<bdi>`, and test it with an
   Arabic or Hebrew name followed by a number.
