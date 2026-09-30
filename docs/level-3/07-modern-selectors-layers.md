---
description: "Modern CSS selectors and cascade control: :is, :where, forgiving lists, :has(), native CSS nesting, and @layer cascade layers, all tested."
---

# 07 · Modern Selectors, Nesting & Cascade Layers

For most of CSS's history, three things required a preprocessor or JavaScript: selecting
an element based on what it *contains*, nesting rules to mirror component structure, and
controlling which stylesheet wins without a specificity arms race. All three are now
native. This lesson covers `:is()`, `:where()` and `:has()`, CSS nesting, and cascade
layers — each tested in Chromium 153.

## `:is()` and `:where()`

Both take a selector list and match any element matching one of them:

```css
:is(h1, h2, h3):hover { color: var(--brand); }
article :is(h2, h3) + p { margin-top: 0; }

/* instead of: */
article h2 + p, article h3 + p { margin-top: 0; }
```

They differ only in specificity (Level 1 · 08):

- `:is()` takes the specificity of its **most specific** argument. `:is(#id, p)` counts
  as an ID even when it matches a `p`.
- `:where()` always has **zero** specificity — ideal for defaults, resets and library
  styles that users should be able to override with any single class.

### Forgiving selector lists

Normally, one invalid selector in a list kills the whole rule. `:is()` and `:where()`
are **forgiving** — invalid arguments are dropped and the rest still work. We tested:

```css
p, p:unknown-thing       { color: rgb(255, 0, 0); }
:is(h1, h1:unknown-thing) { color: rgb(0, 0, 255); }
```

```text
p:  rgb(0, 0, 0)     <- whole rule discarded because of one unknown pseudo-class
h1: rgb(0, 0, 255)   <- :is() ignored the bad argument, kept the good one
```

That matters when mixing new selectors with old: `:is(:focus-visible, .has-focus)` still
works in a browser that doesn't know one of them.

## `:has()` — the relational selector

`:has()` matches an element if the relative selector inside it matches something
**relative to that element** — usually a descendant, but also a following sibling:

```css
.card:has(img)            { padding-top: 0; }                   /* card that contains an image */
.field:has(input:user-invalid) label { color: rgb(176, 0, 32); } /* label of an invalid field */
form:has(#gift:checked) .gift-message { display: block; }        /* show section when box ticked */
h2:has(+ p)               { margin-bottom: 0.25rem; }            /* h2 immediately followed by a p */
body:has(dialog[open])    { overflow: hidden; }                  /* lock scroll while a modal is open */
```

It's often called the "parent selector", but it's more general: it lets any element be
styled based on the state of other elements, which previously needed JavaScript to add
classes.

We tested three of these on a small form. Before interaction:

```text
.extra (the conditional section):  display: none
card with <img>:  padding-top 0px
```

After ticking the checkbox and typing an invalid email then tabbing away:

```text
.extra: display: block
label:  rgb(176, 0, 32)
```

No script; the styles followed the form's state.

Performance note: `:has()` is carefully optimised in current engines, but a broad
selector like `body:has(.x)` or `*:has(...)` means changes deep in the tree can
invalidate styles high up. Anchor `:has()` to the specific component (`.card:has(...)`,
not `:has(...)` on its own) where you can.

## Native CSS nesting

```css
.card {
  padding: 16px;

  & h3 { color: rgb(0, 0, 255); }          /* .card h3 */
  &.featured { border: 2px solid; }        /* .card.featured — no space */
  .dark & { background: rgb(0, 0, 0); }    /* .dark .card — & at the end */
  > .meta { font-size: 0.875rem; }         /* .card > .meta */

  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 0.15); }

  @media (width >= 600px) {                /* nested at-rules apply to .card */
    padding: 32px;
  }
}
```

We checked each variant's result: the nested `h3` rule, the `&.featured` border, the
`.dark &` background and the nested media query (at an 800px viewport, padding was
32px) all applied. A nested rule that starts with a combinator or plain element name
(`> li`, `h3`) is treated as a descendant rule automatically; `&` is only required when
you need to attach something to the parent selector itself (`&.featured`, `&:hover`) or
put the parent somewhere other than the start.

Two differences from Sass-style nesting:

- **No string concatenation.** `&__title` does *not* produce `.card__title` — `&` stands
  for the parent **selector**, not text. BEM-style suffixes need to be written out.
- **Specificity follows `:is()`.** A nested rule behaves like `:is(parent) child`, so a
  parent list like `#main, .card` gives every nested rule ID-level specificity.

Nesting is best kept shallow — one or two levels. Deep nesting recreates the long,
fragile selectors Level 1 · 07 warned about.

## Cascade layers: `@layer`

Specificity fights happen when styles from different sources — a reset, a framework, your
components, one-off utilities — compete. Cascade layers let you declare an order of
**priority between groups of styles**, which is checked **before specificity**:

```css
@layer reset, base, components, utilities;   /* order declared once, first wins lowest */

@layer reset      { *, *::before, *::after { box-sizing: border-box; } }
@layer base       { a { color: var(--link); } }
@layer components { #main .btn { color: rgb(255, 0, 0); } }
@layer utilities  { .text-blue { color: rgb(0, 0, 255); } }
```

We tested three questions about layers:

```css
@layer reset, base, components, utilities;
@layer components { #main .btn { color: rgb(255, 0, 0); } }       /* specificity (1,1,0) */
@layer utilities  { .text-blue { color: rgb(0, 0, 255); } }       /* specificity (0,1,0) */
.btn { background: rgb(1, 1, 1); }                                 /* unlayered */
@layer base       { .btn { background: rgb(2, 2, 2) !important; } }
@layer components { .btn { background: rgb(3, 3, 3) !important; } }
a { color: rgb(0, 128, 0); }                                       /* unlayered, (0,0,1) */
```

```text
button color:      rgb(0, 0, 255)   <- utilities layer beat components despite lower specificity
link color:        rgb(0, 128, 0)   <- unlayered `a` beat layered `#main .btn`
button background: rgb(2, 2, 2)    <- among !important, the EARLIER layer (base) won
```

The three rules of layers:

1. **Later layers beat earlier layers**, regardless of specificity. Specificity only
   decides between rules in the *same* layer.
2. **Unlayered styles beat all layers.** Anything not in a layer is treated as a final,
   implicit layer. This makes adoption easy: put third-party CSS in a low layer and your
   existing styles automatically win.
3. **`!important` reverses layer order** — the earliest layer's important declarations win.
   That lets a reset layer protect something (for example, `[hidden] { display: none
   !important }`) that later layers can't accidentally override.

### Putting libraries in a layer

```css
@import url("vendor/datepicker.css") layer(vendor);
@layer vendor, base, components, utilities;
```

Now nothing in the date-picker's CSS can beat your components, however specific its
selectors are. And you override it with plain class selectors instead of fighting its
`#datepicker .dp-table td.dp-day` chains.

Layers can nest (`@layer components.buttons`) and be revisited — adding more rules to an
existing layer later in the file keeps the layer's original position. Declare the order
once at the top of your main stylesheet so it's explicit.

## Worked example: a component stylesheet using all three

```css title="components/alert.css"
@layer components {
  .alert {
    --accent: var(--color-info);
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.75rem;
    padding: 1rem;
    border-inline-start: 4px solid var(--accent);
    background: color-mix(in oklch, var(--accent) 8%, var(--color-bg));

    &.is-warning { --accent: var(--color-warning); }
    &.is-error   { --accent: var(--color-error); }

    &:not(:has(> svg)) { grid-template-columns: 1fr; }   /* no icon column if no icon */

    & :where(h2, h3) { margin: 0; font-size: 1rem; }      /* zero-specificity defaults */

    & a { color: inherit; font-weight: 600; }
  }
}
```

Every piece is doing a job: the layer keeps component styles below utilities, nesting
groups the variants, `:has()` adapts the layout to the content, and `:where()` keeps the
heading defaults trivially overridable.

## How It Actually Works

The cascade (Level 1 · 08) sorts declarations by origin and importance, then context,
then the style attribute, then **layer order**, then specificity, then source order.
Layers slot in above specificity. Internally each declaration carries its layer's index;
comparing two declarations checks that index before computing specificity at all.

`:has()` required new invalidation machinery. Normally, when an element changes, engines
only restyle that element and its descendants (and siblings for `+`/`~`). With `:has()`,
a change to a descendant can change whether an **ancestor** matches. Engines handle this
by recording, at selector-parse time, which features inside `:has()` arguments could
matter (classes, attributes, pseudo-classes like `:checked`) and, when one of those
changes on an element, walking *up* to find ancestors whose `:has()` rules need
rechecking. That's why narrowly anchored `:has()` selectors are cheaper.

Nesting is purely syntactic: the parser expands nested rules into full selectors, with
the parent wrapped as `:is(parent)`. Nothing about matching or the cascade is new.

## Common mistakes

- **Assuming `:is()` has zero specificity** — that's `:where()`.
- **Sass habits in native nesting** — `&-suffix` concatenation doesn't work.
- **Deep nesting** producing long selectors.
- **Unanchored `:has()`** selectors on large, frequently changing pages.
- **Being surprised that unlayered CSS beats layered CSS** when you adopt layers partway.
- **Forgetting that `!important` inverts layer order.**

## Exercise

1. Replace three repeated selector lists in your stylesheets with `:is()`, and wrap your
   reset in `:where()` so it has zero specificity.
2. Use `:has()` to show a "gift message" textarea only when a "This is a gift" checkbox
   is ticked, and to highlight a whole `.field` when its input is `:user-invalid`.
3. Rewrite one component's CSS using native nesting, no more than two levels deep.
4. Organise your whole stylesheet into `reset, base, layout, components, utilities`
   layers. Add a deliberately over-specific rule in `components` and confirm a single-class
   utility still overrides it.
5. Recreate the `!important` layer test and explain the result to someone else.
