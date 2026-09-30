---
description: "How CSS picks the winning declaration: origins and !important, specificity by hand, source order, inheritance, and inherit/initial/revert."
---

# 08 · The Cascade, Specificity & Inheritance

Sooner or later every CSS author writes a rule, reloads, and sees nothing change. The
rule is fine; another rule is winning. The algorithm that picks the winner is **the
cascade** — the "C" in CSS — and it's completely deterministic. Once you can run it in
your head, "why isn't my style applying?" becomes a two-minute question.

## The problem the cascade solves

For any element and any property, many declarations may apply: the browser's defaults,
your stylesheet, a component library, an inline `style`. Each element ends up with
**exactly one value per property**. The cascade sorts the competing declarations and
takes the first one, comparing these criteria **in order**, moving to the next only on a
tie:

1. **Origin and importance** — who wrote it, and is it `!important`?
2. **Context** — declarations inside a shadow DOM (web components) vs outside. You can
   ignore this until you write web components.
3. **Style attribute** — `style="..."` beats selector-based rules of the same origin.
4. **Cascade layers** — `@layer` order (Level 3 · 07).
5. **Specificity** — how specific the selector is.
6. **Order of appearance** — later wins.

In day-to-day work, it's almost always **specificity** or **order** deciding.

## Origins and `!important`

There are three origins: the **user-agent** stylesheet (browser defaults), **user**
styles (a person's own preferences, e.g. forced larger fonts), and **author** styles
(yours). For normal declarations, author beats user beats user-agent. `!important`
**reverses** that order for important declarations:

| Precedence (highest first) |
|---------------------------|
| Transitions in progress |
| User-agent `!important` |
| User `!important` |
| Author `!important` |
| Animations (`@keyframes`) |
| Author normal |
| User normal |
| User-agent normal |

The reversal exists so that a user who needs, say, high-contrast colours can always
override the site. It also means `!important` in your own stylesheet is a sledgehammer:
it jumps above *every* normal author declaration regardless of specificity. We confirmed
it in Chromium:

```css
#hero .title { color: rgb(255, 0, 0); }          /* specificity (1,1,0) */
header h1.title { color: rgb(0, 0, 255); }       /* (0,1,2) */
.title { color: rgb(0, 128, 0) !important; }    /* (0,1,0) but important */
```

```text
computed color of h1: rgb(0, 128, 0)
```

The weakest selector won because importance is compared **before** specificity.

## Specificity

Specificity is a three-part score written **(A, B, C)**:

- **A** — number of **ID** selectors.
- **B** — number of **class** selectors, **attribute** selectors and **pseudo-classes**.
- **C** — number of **type** (element) selectors and **pseudo-elements**.

The universal selector `*` and combinators (` `, `>`, `+`, `~`) add nothing.

Compare column by column, left to right; the first column that differs decides. **Any
number of classes can never beat a single ID**, because the A column is compared first:

| Selector | A | B | C | Score |
|----------|---|---|---|-------|
| `p` | 0 | 0 | 1 | (0,0,1) |
| `article p` | 0 | 0 | 2 | (0,0,2) |
| `.note` | 0 | 1 | 0 | (0,1,0) |
| `p.note` | 0 | 1 | 1 | (0,1,1) |
| `a:hover` | 0 | 1 | 1 | (0,1,1) |
| `input[type="email"]` | 0 | 1 | 1 | (0,1,1) |
| `.card .title::before` | 0 | 2 | 1 | (0,2,1) |
| `header h1.title.big.bold` | 0 | 3 | 2 | (0,3,2) |
| `#hero .title` | 1 | 1 | 0 | (1,1,0) |
| `style="..."` | — | — | — | beats all selectors (step 3 above) |

We tested the "classes can't beat an ID" rule:

```css
#hero .title { color: rgb(255, 0, 0); }               /* (1,1,0) */
header h1.title.big.bold { color: rgb(0, 0, 255); }   /* (0,3,2) */
```

```text
computed color of h1: rgb(255, 0, 0)
```

### Special pseudo-classes

- `:is(...)`, `:not(...)` and `:has(...)` take the specificity of their **most specific
  argument**. `:not(#x)` counts as an ID.
- `:where(...)` always has **zero** specificity. That makes it perfect for defaults that
  should be easy to override:

```css
.btn { color: rgb(255, 0, 0); }
.btn { color: rgb(0, 0, 255); }
:where(#main) .btn { color: rgb(0, 128, 0); }   /* (0,1,0): the #main counts for nothing */
```

```text
computed color: rgb(0, 128, 0)
```

All three rules tie at (0,1,0), so **order of appearance** decides, and the last one
wins. Without the `:where()` wrapper, `#main .btn` would have won on specificity alone —
here it won only because it came last.

## Order of appearance

When everything else ties, the declaration that appears **later** wins. "Later" is
across the whole document: stylesheets are concatenated in the order they're linked,
and `<style>` blocks count at their position. That's why you load a library's CSS
*before* your own.

## Inheritance

Some properties **inherit**: if an element has no value from the cascade, it takes its
parent's computed value. Roughly, text-related properties inherit and box-related ones
don't:

| Inherit by default | Don't inherit |
|--------------------|---------------|
| `color`, `font-*`, `line-height`, `letter-spacing`, `text-align`, `text-indent`, `white-space`, `visibility`, `cursor`, `list-style`, `direction`, custom properties | `margin`, `padding`, `border`, `background`, `width`, `height`, `display`, `position`, `overflow`, `opacity` |

That's what lets you set `font-family` once on `body`. We set colour, font and a border on
`body` and checked a paragraph inside it:

```css
body { color: rgb(50, 50, 50); font-family: Georgia; border: 2px solid red; }
a { color: inherit; }
button { font: inherit; }
```

```text
p color: rgb(50, 50, 50)     <- inherited
p border style: none         <- border doesn't inherit
a color: rgb(50, 50, 50)     <- only because of `color: inherit`
button font-family: Georgia  <- only because of `font: inherit`
```

Links and form controls are the exceptions you'll fight with. Links get their blue from
the user-agent stylesheet (a cascaded value beats inheritance), and buttons and inputs
have their own font settings. Without our rule, Chromium's defaults for both were:

```text
button: 13.3333px Arial
input:  13.3333px Arial
```

That's why almost every reset stylesheet includes `button, input, select, textarea {
font: inherit; }`.

**Inheritance has no specificity.** An inherited value loses to *any* declaration that
targets the element directly — even `* { color: black; }`. If you set `color` on a
parent and the child ignores it, look for a rule targeting the child.

## The global keywords

Every property accepts these values:

| Keyword | Meaning |
|---------|---------|
| `inherit` | Use the parent's computed value |
| `initial` | Use the property's **spec** initial value (e.g. `display: initial` is `inline`, even on a `div`) |
| `unset` | `inherit` for inherited properties, `initial` for the rest |
| `revert` | Roll back to the value the previous origin would give — usually the browser default |
| `revert-layer` | Roll back to the previous cascade layer |

In our test, `.unset { border: unset; color: unset; }` produced a colour of
`rgb(50, 50, 50)` (inherited from `body`) and a border width of `0px` (initial). The
common trap is `initial` on `display`: `div { display: initial }` makes the div
**inline**, because the spec's initial value is `inline` — the `block` you're used to
comes from the user-agent stylesheet. Use `revert` to get the browser default back.

## Worked example: debugging a style that won't apply

```html
<nav id="site-nav">
  <a class="nav-link active" href="/">Home</a>
</nav>
```

```css
#site-nav a { color: #333; }        /* (1,0,1) */
.nav-link.active { color: #b3401a; } /* (0,2,0) — loses */
```

The active link stays grey. In devtools, the `.nav-link.active` colour is struck through,
with `#site-nav a` listed above it. Options, from best to worst:

1. **Lower the other rule's specificity** — `.site-nav a` or `:where(#site-nav) a`. Now
   `.nav-link.active` wins by specificity.
2. **Raise yours just enough** — `#site-nav .active` (1,1,0) beats (1,0,1).
3. **`!important`** — works, but now the next override also needs `!important`.

Option 1 fixes the cause: an ID in a styling selector.

## How It Actually Works

The cascade is one stage in computing each property's value, which the spec describes as
a pipeline:

1. **Declared values** — every declaration that applies to the element.
2. **Cascaded value** — the winner of the sort described above (or nothing).
3. **Specified value** — the cascaded value, or if there isn't one, the inherited value
   for inherited properties and the initial value for the rest ("defaulting").
4. **Computed value** — relative values resolved as far as possible without layout:
   `2em` becomes pixels, `inherit` becomes the parent's value, colour keywords become
   `rgb()`. **This** is what inherits.
5. **Used value** — after layout: percentages of widths become pixels.
6. **Actual value** — after rounding to what the device can display.

`getComputedStyle()` returns what the spec calls the *resolved value* — usually the
computed value, but the used value for some layout properties like `width`. The devtools
Computed pane shows the same thing, and clicking the arrow next to a property shows every
declaration that competed for it.

Because computed values are what inherit, `font-size: 2em` on a parent passes down an
absolute pixel size, not "2em" — children don't double again unless they also declare
`em`s. Lesson 09 shows what happens when they do.

## Common mistakes

- **Reaching for `!important`** instead of finding the competing rule.
- **Styling with IDs**, then needing more IDs or `!important` to override.
- **Expecting inherited values to beat direct rules**, like `color` on a parent vs a
  user-agent rule on `a`.
- **`initial` when you mean "browser default"** — use `revert`.
- **Relying on source order across files** you don't control the load order of.
- **Long selectors "to be safe"**, raising specificity and making every later override
  harder.

## Exercise

1. Calculate the specificity of: `ul li a`, `.menu > li:first-child a`, `#nav .item`,
   `a[href^="https"]:hover`, `:is(#a, .b) p`, `:where(#a, .b) p`. Check your answers by
   pasting each into a devtools Styles pane and hovering the selector (Chrome shows the
   specificity).
2. Recreate the "debugging" example and fix it all three ways. Which fix survives adding
   a new `.nav-link.hover` state later?
3. Set `font-family` and `color` on `body`. Find every element on your recipe page that
   *doesn't* pick them up and explain why for each.
4. Put `display: initial` on a `<div>` and observe it; then change it to `revert`.
