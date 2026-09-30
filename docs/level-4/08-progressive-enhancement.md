---
description: "Use new HTML and CSS safely: how browsers skip unknown CSS, fallbacks, @supports, Baseline, vendor prefixes and graceful degradation."
---

# 08 · Progressive Enhancement & Browser Support

Browsers update every few weeks, and CSS gains new features every year — this course has
used several that arrived recently: `:has()`, container queries, `@layer`, subgrid,
`light-dark()`, anchor positioning, `@starting-style`. Your users, though, don't all run
the newest browser. Some are on older phones that no longer get updates, on locked-down
corporate machines, or on niche browsers. **Progressive enhancement** is the strategy that
lets you use new features without breaking the page for them: build a baseline that works
everywhere, then layer improvements on top for browsers that support them.

## The fundamental rule: unknown CSS is ignored

CSS was designed to be forward-compatible. A browser that doesn't understand a
declaration skips **just that declaration**; one that doesn't understand a selector skips
**that rule**; one that doesn't understand an at-rule skips **that block**. Nothing
crashes. That makes fallbacks trivial — declare the safe value first, the new one second:

```css
.a { display: block; display: grid-lanes-not-real; }                   /* unknown value */
.b { background: rgb(200, 0, 0); background: color-mix(in oklch, red, blue); }
.c { color: rgb(0, 0, 0); color: nonsense(1); }                         /* unknown function */
```

We rendered those in Chromium:

```text
.a display:          block                                  <- the unknown value was dropped
.b background-color: oklch(0.539974 0.285457 326.643)       <- supported, so it won
.c color:            rgb(0, 0, 0)                           <- the invalid second value was dropped
```

Each browser keeps the last declaration it understands. An old browser without
`color-mix()` would render `.b` red; a new one gets the mixed colour.

**One exception to remember** (Level 2 · 09): declarations using `var()` can't be checked
at parse time, so the fallback-first pattern doesn't protect you from an invalid *variable
value*. And a comma-separated selector list is all-or-nothing (Level 3 · 07) — unless you
use `:is()`/`:where()`.

## `@supports`: feature queries

When an enhancement needs several declarations together, or needs to *undo* the
fallback, wrap it in a feature query:

```css
/* baseline: a flexbox gallery that wraps */
.gallery { display: flex; flex-wrap: wrap; gap: 1rem; }
.gallery > * { flex: 1 1 15rem; }

/* enhancement: subgrid-aligned cards */
@supports (grid-template-rows: subgrid) {
  .gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(15rem, 1fr)); }
  .gallery > * { display: grid; grid-row: span 3; grid-template-rows: subgrid; }
}

/* selector support */
@supports selector(:has(a)) {
  .card:has(img) { padding-top: 0; }
}

/* the negative form, for fallbacks only */
@supports not (container-type: inline-size) {
  .card { max-width: 40rem; }
}
```

We tested three queries in Chromium; all applied their rules:

```text
@supports (container-type: inline-size)  -> matched
@supports not (display: nonsense)        -> matched (the value isn't supported)
@supports selector(:has(a))              -> matched
```

JavaScript has the same test: `CSS.supports('anchor-name: --x')` or
`CSS.supports('selector(:has(a))')` — this course used it to report feature support for
each engine we tested.

Caveat: `@supports` tests whether the browser can **parse** a declaration, not whether it
implements it fully or bug-free. It's excellent for "does this feature exist" and useless
for "does this feature work correctly with X."

## HTML degrades gracefully too

HTML's error handling (Level 1 · 01) makes new elements and attributes safe in the same
way:

- **Unknown elements** render as inline generic elements — so `<search>` in an old
  browser is just a container (you lose the landmark, not the content).
- **Unknown attributes** are ignored — `popover`, `loading="lazy"`, `fetchpriority`,
  `inert` simply do nothing in browsers that don't know them. Design so that "nothing"
  is acceptable: a `popover` element in a browser without popover support is displayed
  inline (you may want to hide it via `@supports` in that case), and `loading="lazy"`
  falls back to eager loading.
- **Unknown input types** become `type="text"` — a `type="date"` in a browser without a
  date picker is a text box, so accept typed dates on the server.

## Baseline: knowing what's safe

**Baseline** is a cross-browser-vendor project (run with the W3C WebDX community group)
that labels web features:

- **Baseline "newly available"** — supported in the current versions of Chrome, Edge,
  Firefox and Safari (desktop and mobile).
- **Baseline "widely available"** — has been newly available for 30 months, so it's safe
  for the large majority of users without fallbacks.
- **Limited availability** — not yet in all of them.

MDN and caniuse.com display Baseline status on each feature. A practical policy:

- **Widely available** → use freely.
- **Newly available** → use, with a fallback that keeps the page usable.
- **Limited availability** → use only as an enhancement behind `@supports` or with a
  harmless no-op fallback.

For precise numbers for *your* audience, check your analytics — a site used mainly by
developers and one used mainly on older Android phones have very different browser mixes.

## Vendor prefixes

In the past, browsers shipped experimental CSS with prefixes (`-webkit-`, `-moz-`,
`-ms-`). Modern browsers ship new features behind flags instead, so you rarely need to
write prefixes by hand. A few survive as the de-facto standard (`-webkit-line-clamp`,
`-webkit-text-stroke`, `::-webkit-details-marker`), and a few properties still need a
prefix for some Safari versions. Rather than remembering which, use a build tool:
**Autoprefixer** (a PostCSS plugin) or **Lightning CSS** add exactly the prefixes needed
for your declared browser targets (a `browserslist` query like
`"> 0.5%, last 2 versions, not dead"`).

## Worked example: an enhanced popover menu

The Level 3 popover with anchor positioning, written to degrade in three stages:

```html
<div class="menu">
  <button type="button" popovertarget="account-menu" class="menu__button">Account</button>
  <ul id="account-menu" popover class="menu__list">
    <li><a href="/profile">Profile</a></li>
    <li><a href="/orders">Orders</a></li>
    <li><a href="/sign-out">Sign out</a></li>
  </ul>
</div>
```

```css
/* Stage 0 — no popover support: the list is shown inline as normal links.
   Hide the useless button, keep the links. */
.menu__button { display: none; }

@supports selector(:popover-open) {
  /* Stage 1 — popover supported: button shows, list is a centred popover */
  .menu__button { display: inline-block; }
  .menu__list { margin: auto; padding: 0.5rem; list-style: none; }

  @supports (anchor-name: --a) {
    /* Stage 2 — anchor positioning: list attaches to its button */
    .menu__button { anchor-name: --account; }
    .menu__list {
      position-anchor: --account;
      inset: auto;
      margin: 0.25rem 0 0;
      position-area: bottom span-left;
      position-try-fallbacks: flip-block;
    }
  }
}
```

In the oldest browsers users get a plain list of links (fully functional); with popover
support, a dropdown centred on screen; with anchor positioning, a dropdown attached to its
button. At no stage is anything broken.

We checked stage 2 in Chromium: after clicking the button (bottom edge at y=229, right
edge at x=272), the list opened with its top at 233 and its right edge at 272 — directly
below the button, aligned to its right edge as `span-left` asks, with the 0.25rem gap.

## How It Actually Works

CSS error handling is part of the syntax specification. The parser first tokenises the
stylesheet and groups tokens into rules and declarations by braces and semicolons —
without knowing what any property means. Only then does each declaration go to the
property-specific grammar: if it doesn't match (unknown property, unknown value, wrong
type), it's discarded **individually**. Because the grouping step doesn't depend on
understanding the content, a browser always knows where an unknown rule ends, so it can
skip exactly that rule and carry on.

`@supports (property: value)` runs the same property grammar in test mode and returns
whether it would have been accepted. `selector()` runs the selector parser. Neither
checks rendering behaviour, which is why partial or buggy implementations can pass.

## Common mistakes

- **Using a new feature with no fallback**, so older browsers get a broken layout rather
  than a simpler one.
- **Fallback declared after the new value**, so it always wins.
- **Using `@supports not`** for enhancements (it inverts the logic and misfires in
  browsers that don't support `@supports` itself — rare now, but a sign of fragile code).
- **Assuming `@supports` means "works"** for complex features.
- **User-agent sniffing** instead of feature detection.
- **Hand-written vendor prefixes** that go stale.

## Exercise

1. List every newer feature your projects use (subgrid, `:has()`, container queries,
   `light-dark()`, anchor positioning, `@starting-style`...) and look up each one's
   Baseline status on MDN.
2. For each "newly available" or "limited" feature, write down what an unsupporting
   browser shows. Add fallbacks where the answer is "something broken."
3. Build the three-stage popover menu. Test stage 1 by commenting out the anchor block.
   Stage 0 can't be fully reproduced in a browser that supports popovers: renaming
   `popover` to `data-popover` shows the list inline (as an old browser would), but the
   button still appears because `@supports` still matches. Temporarily change the
   selector in the `@supports` condition to something unsupported to see the full stage 0.
4. Add Autoprefixer or Lightning CSS to a project with a `browserslist` setting and
   diff the output CSS against your source.
