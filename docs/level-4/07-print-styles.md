---
description: "Style pages for print: @media print, hiding page chrome, printing link URLs, page breaks, @page size and margins, and testing print output."
---

# 07 · Print Styles & Other Media

People still print web pages — recipes to take into the kitchen, boarding passes,
invoices, directions, articles to read later, forms for signing — and far more "print"
to PDF to save or share. Without a print stylesheet, they get navigation menus, cookie
banners, dark backgrounds that waste ink, and headings stranded at the bottom of a page.
A few dozen lines of CSS fix all of it.

## Print styles live in a media query

```css
@media print {
  /* rules that only apply when printing or saving as PDF */
}
```

Or in a separate file, which isn't render-blocking for the screen:

```html
<link rel="stylesheet" href="/css/print.css" media="print">
```

## What a print stylesheet should do

```css title="css/print.css"
@media print {
  /* 1. Remove what's useless on paper */
  nav, .site-header nav, .skip-link, .ad, .cookie-banner, .share-buttons,
  video, audio, button, .no-print {
    display: none !important;
  }

  /* 2. Readable, ink-friendly text */
  body {
    font: 12pt/1.4 Georgia, "Times New Roman", serif;
    color: black;
    background: white;
  }
  main { width: auto; margin: 0; padding: 0; }   /* drop screen max-widths */

  /* 3. Make links useful on paper */
  a { color: inherit; text-decoration: underline; }
  a[href^="http"]::after { content: " (" attr(href) ")"; font-size: 0.85em; }
  a[href^="#"]::after, a[href^="javascript:"]::after { content: ""; }

  /* 4. Control page breaks */
  h1, h2, h3 { break-after: avoid; }            /* keep headings with what follows */
  figure, table, pre, blockquote, .recipe-card { break-inside: avoid; }
  p { orphans: 3; widows: 3; }                  /* at least 3 lines at page top/bottom */
  .chapter { break-before: page; }

  /* 5. Show things that are collapsed on screen */
  details::details-content { content-visibility: visible; display: block; }

  img { max-width: 100% !important; }
}
```

Units: `pt`, `cm`, `mm` and `in` are the natural units for paper. `12pt` is 16px — the
same size the screen uses by default, just in print terminology.

The `!important` in the hiding rule is one of the legitimate uses: print overrides should
beat every component rule, whatever its specificity (a cascade layer placed last would
work too).

## Measured: screen vs print

We loaded a page in Chromium and read computed styles under the screen media type, then
switched Playwright to emulate `print` and read them again. The page's stylesheet
contained the rules above for `nav`, `.ad`, the body font and the external-link
`::after`:

```text
screen: nav display "block", .ad display "block", body font "Times"
print:  nav display "none",  .ad display "none",  body font "Georgia, serif",
        link ::after content "\" (https://example.com/basil)\""
```

The link's URL is printed after it, so "See basil guide (https://example.com/basil)"
still makes sense on paper. (The screen font is "Times" only because this test page set
no screen font.)

### Printing closed `<details>`

Rule 5 needed a second attempt. Our first version was `details > * { display: block; }`,
a pattern that's been passed around for years. We checked it in Chromium and WebKit with
`checkVisibility()` on the content of a closed `<details>`: it returned `false` in both —
the content stayed hidden, because browsers hide it via an internal slot, not the
children's `display`. The `::details-content` pseudo-element targets that slot directly;
with the rule above, `checkVisibility()` returned `true` in both engines and the content
took up space. `::details-content` is recent, so in browsers without it the content
simply stays collapsed. If printing it is essential, also open every `<details>` from a
`beforeprint` event listener in JavaScript.

## `@page`: paper size and margins

```css
@page {
  size: A4;          /* or letter, or 210mm 297mm, or A4 landscape */
  margin: 2cm;
}

@page :first {
  margin-top: 3cm;   /* a larger top margin on the first page */
}
```

`@page` rules apply to the page boxes the content flows into. We generated a PDF from a
long test page with `size: A4; margin: 2cm` using Chromium's print-to-PDF (with "prefer
CSS page size" enabled, as the print dialog does by default) and inspected the file:

```text
pages: 7   first MediaBox: 0 0 594.95996 841.91998   (points)
```

594.96 × 841.92 points is A4 (210 × 297 mm at 72 points per inch). The page count
depends on content, fonts and margins, so yours will differ.

Browsers also support page-margin boxes for running headers and footers (`@top-center`,
`@bottom-right` with `content: counter(page)`) to varying degrees; check your target
browsers before relying on them, and remember that the print dialog's own "headers and
footers" option adds the URL and date independently of your CSS.

## Colour and backgrounds

Browsers don't print background colours and images by default (to save ink) unless the
user enables "Background graphics". Two consequences:

- **Don't rely on backgrounds for meaning** — a status shown only as a green background
  vanishes on paper. Use text, or borders, which do print.
- **To force a background** where it genuinely matters (a colour legend, a QR code panel):

  ```css
  .legend-swatch { print-color-adjust: exact; }
  ```

## Designing specific printable documents

Some pages are *made* to be printed — invoices, tickets, certificates, labels. Treat these
as a print layout first:

```css
@media print {
  @page { size: A4; margin: 15mm; }

  .invoice { display: grid; grid-template-columns: 1fr auto; gap: 8mm; }
  .invoice__lines { width: 100%; border-collapse: collapse; }
  .invoice__lines thead { display: table-header-group; }   /* repeat header on every page */
  .invoice__lines tr { break-inside: avoid; }
  .invoice__total { break-before: avoid; }
}
```

`display: table-header-group` on `<thead>` is the default, but some resets change it; it's
what makes table headers repeat at the top of each printed page.

## Other media types and features

`print` and `screen` are the two media types in practical use (`all` matches everything;
older types like `handheld` and `tv` are deprecated). Media *features* that matter beyond
width, covered in Level 2 · 08, include:

| Feature | Use |
|---------|-----|
| `prefers-color-scheme` | Dark mode |
| `prefers-reduced-motion` | Motion sensitivity |
| `prefers-contrast` | Users asking for more (or less) contrast |
| `forced-colors: active` | Windows High Contrast / forced colours mode |
| `hover`, `pointer`, `any-pointer` | Input capabilities |
| `resolution` | High-DPI tweaks (`(resolution >= 2dppx)`) |
| `scripting: none` | JavaScript disabled |
| `display-mode: standalone` | Installed web app |

### Forced colours

In forced-colours mode, the browser replaces your colours with a small user-chosen
palette. Backgrounds, shadows and gradients disappear; text, borders and outlines are
recoloured. Make sure important boundaries are **borders or outlines**, not background
changes, and use system colour keywords if you need to adjust:

```css
@media (forced-colors: active) {
  .button { border: 1px solid ButtonText; }
  .card:hover { outline: 2px solid Highlight; }
  svg { fill: CanvasText; }
}
```

## Worked example: a printable recipe

```css
@media print {
  .site-header, .site-footer, .toc, .related, .comments, .skip-link { display: none !important; }

  body { font: 11pt/1.45 Georgia, serif; color: black; background: none; }

  .recipe { columns: 2; column-gap: 1cm; }                 /* ingredients and method side by side */
  .recipe h1, .recipe .facts, .recipe figure { column-span: all; }
  .recipe figure img { max-height: 6cm; width: auto; }     /* smaller photo on paper */

  .steps li { break-inside: avoid; }
  .facts div { border: 1pt solid #999; }                   /* borders print, backgrounds don't */

  .recipe::after {
    content: "From Weeknight Kitchen — " attr(data-url);
    display: block;
    margin-top: 1cm;
    font-size: 9pt;
    color: #555;
  }
}
```

Most recipes fit on one sheet: a two-column layout, a smaller photo, no navigation or
comments, and the source URL printed at the end (`data-url` attribute on the article).

## Testing print styles

- **Print preview** (Ctrl/Cmd+P) in each browser — they differ in page-break behaviour and
  `@page` support.
- **Devtools emulation**: Chrome/Edge devtools → Rendering → "Emulate CSS media type:
  print" shows print styles in the normal viewport, where you can inspect them.
- **Automated**: Playwright's `page.emulateMedia({ media: 'print' })` and `page.pdf()`
  (Chromium) produce the output we measured above, and can be diffed in CI.

## How It Actually Works

When printing, the browser re-runs style and layout with the `print` media type active
and a **paged** context: instead of one infinitely tall viewport, content is fragmented
into a sequence of page boxes sized by `@page` (or the printer's default). **Fragmentation**
is the layout process of breaking boxes across pages (the same machinery handles CSS
multi-column layout). Break properties (`break-before`, `break-after`, `break-inside`,
`orphans`, `widows`) are hints to that algorithm: the browser prefers break points that
satisfy them, but will break anyway if something is taller than a page.

Print layout is separate from the screen layout — elements hidden only in print keep
working on screen, and media queries based on width see the printable area's width
rather than the window's.

## Common mistakes

- **No print styles at all**, printing navigation, ads and banners.
- **Link text with no URL** on paper.
- **Meaning carried by background colour.**
- **Fixed pixel widths** that overflow the printable area.
- **Headings orphaned at the bottom of a page.**
- **Content in collapsed `<details>` or inactive tabs** that never prints.
- **Testing only in one browser's print preview.**

## Exercise

1. Write a print stylesheet for your recipe page and check it in print preview in two
   browsers. Aim for one A4 page.
2. Use devtools' print media emulation to inspect your print styles, then print to PDF.
3. Make an invoice page with a table long enough to span two pages, and make the header
   row repeat on each page.
4. Turn on Windows High Contrast (or emulate `forced-colors: active` in Chrome devtools'
   Rendering panel) and check your Level 3 gallery. Fix any control whose boundary
   disappears.
